**AV1 QSV - Guia prático para Intel Arc no Linux.**

Um guia direto ao ponto para extrair o máximo do AV1 usando Intel Arc via QSV no Linux,
tudo aqui é baseado em testes reais, uso no dia a dia e ajustes práticos para FFmpeg 7+,
a ideia é simples, boa qualidade e bom desempenho sem ficar perdido na documentação oficial.

**Sistema de testes:**

Todo o comportamento descrito neste guia foi testado no ambiente abaixo.

**Distro:** Fedora 43 KDE

**Kernel:** 6.14.11

**GPU:** Intel Arc A310

**CPU:** Ryzen 5 4600G

**FFmpeg:** 7.1.2

Driver stack: Intel Media Driver (iHD) + OneVPL


**Caminhos de exemplo:**

Os caminhos usados nos comandos (/run/media/malk/Downloads/input.mkv etc.) são exemplos reais do meu setup,
Mantenha as aspas e ajuste os caminhos conforme o seu ambiente.

**Comandos base:**

Abaixo estão os dois comandos base que uso no dia a dia,
eles cobrem fontes AVC, HEVC e AV1, a única diferença entre eles é a profundidade de bits da fonte.

Para fontes AVC, HEVC e AV1 8 bits:

```bash
ffmpeg \
 -i "/run/media/malk/Downloads/input.mkv" \
 -map 0:v:0 \
 -vf "format=p010le" \
 -pix_fmt p010le \
 -c:v av1_qsv \
   -preset veryslow \
   -global_quality 24 \
   -look_ahead_depth 100 \
   -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 8 \
   -extbrc 1 -g 300 -forced_idr 1 \
   -tile_cols 0 -tile_rows 0 \
 -an \
 "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv"
```


Para fontes AVC, HEVC e AV1 10 bits:

```bash
ffmpeg \
 -i "/run/media/malk/Downloads/input.mkv" \
 -map 0:v:0 -c:v av1_qsv \
   -preset veryslow \
   -global_quality 24 \
   -look_ahead_depth 100 \
   -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 8 \
   -extbrc 1 -g 300 -forced_idr 1 \
   -tile_cols 0 -tile_rows 0 \
  -an \
 "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv"
```

**Nota sobre profundidade de bits e decodificação:**

Em fontes 8 bits faço conversão explícita para 10 bits usando tanto -vf "format=p010le" quanto -pix_fmt p010le para garantir conversão explícita e evitar qualquer negociação automática para 8 bits, isso reduz banding e melhora a estabilidade visual no AV1, já em fontes HEVC 10 bits esse é o único cenário onde uso decodificação acelerada via QSV, todo o restante foi testado e se mostrou incompatível ou instável, por isso AVC e outras fontes ficam sempre em decodificação por software.


Explicação dos parâmetros utilizados.


Qualidade e controle:


**-preset veryslow**

Uso o preset veryslow porque foi onde o AV1_QSV da Arc A310 realmente começou a extrair eficiência de compressão, presets mais rápidos funcionam mas desperdiçam bitrate em cenas simples e quebram com mais facilidade em cenas complexas, nos testes esse preset entregou o melhor equilíbrio entre tempo de encode e qualidade final.

**-global_quality 24**

Uso o **-global_quality 24** como ponto base porque foi onde consegui manter os arquivos dentro do meu teto de tamanho sem introduzir artefatos evidentes em anime, na prática eu trato cada episódio individualmente e ajusto se necessário, valores mais baixos aumentam demais o tamanho e valores mais altos começam a destruir gradientes, linhas finas e texturas, esse intervalo foi validado episódio por episódio nos testes reais.

**-look_ahead_depth 100**

Defini **-look_ahead_depth 100** porque valores menores fazem o encoder tomar decisões ruins em cenas com cortes rápidos ou mudanças bruscas de complexidade, valores maiores não trouxeram ganho visual proporcional e só aumentaram o custo, esse foi o ponto onde o rate control passou a se comportar de forma previsível.

**-extbrc 1**

Ativei **-extbrc** porque o controle de bitrate padrão do QSV reage mal a cenas difíceis, com extbrc ligado o encoder responde melhor a variações bruscas de complexidade, isso ficou claro em cenas escuras com grão e efeitos.


**Estrutura temporal:**


**-adaptive_i 1**

Ativei adaptive_i porque sem isso o QSV tende a inserir I-frames de forma desnecessária, com esse parâmetro o encoder só usa I-frame quando há quebra real de continuidade visual, nos testes isso reduziu picos de bitrate inúteis.

**-adaptive_b 1**

Uso **-adaptive_b** porque B-frames fixos não funcionam bem em todo tipo de cena, principalmente em anime com longos trechos estáticos seguidos de movimento rápido, com isso ativo o encoder decide melhor quando usar ou não B-frames.

**-b_strategy 1**

Mantenho **-b_strategy** ligado porque ele permite reorganizar melhor a hierarquia dos B-frames, sem isso a distribuição fica engessada, nos testes o resultado prático foi menos flicker e melhor preservação de detalhes em movimento.

**-bf 8**

Escolhi **-bf 8** porque foi o máximo que trouxe ganho real de compressão sem introduzir instabilidade visual, valores menores desperdiçam potencial do AV1 e valores maiores não mostraram diferença prática nos testes.

**-g 300**

Uso **-g 300** porque anime se beneficia muito de GOP longo, há muita reutilização de informação entre frames, valores menores quebraram eficiência sem trazer ganho real em compatibilidade ou seek.

**-forced_idr 1**

Ativei **-forced_idr** para garantir pontos limpos de recuperação no stream, isso evita problemas de seek e reprodução em players mais sensíveis, nos testes isso foi mais confiável do que deixar tudo automático.


**Layout e pipeline:**


**-tile_cols 0 -tile_rows 0**

Mantenho tiles explicitamente desativados porque em 1080p e 1440p eles não trouxeram ganho visual nem de compressão, apenas aumentaram overhead, o valor 0 é uma escolha ativa baseada em testes.

**-an**

Removo o áudio nesse estágio porque meu foco aqui é validar exclusivamente o vídeo, isso reduz variáveis e acelera testes, o áudio é tratado depois em um passo separado com Opus.



**Parâmetros existentes e compatíveis que eu não uso, e o porquê.**


**Rate control e bitrate:**


**-rc cbr**

Não uso **-rc cbr** porque ele força bitrate constante mesmo quando a cena não precisa, em anime isso desperdiça bits em cenas simples e ainda falha em cenas complexas, o tamanho fica previsível mas a qualidade não

**-rc vbr**

Não uso **-rc vbr** porque apesar de parecer flexível ele ainda tenta seguir metas de bitrate, com teto rígido de tamanho isso ficou menos previsível que **-global_quality**, em vários episódios reagiu tarde demais a picos de complexidade

**-rc icq**

Não uso **-rc icq** porque apesar de funcionar ele se mostrou menos consistente entre episódios do que **-global_quality**, principalmente em cenas escuras, comparei diretamente os dois e mantive o que foi mais previsível no meu cenário

**-maxrate**

Não uso **-maxrate** porque esse parâmetro é voltado para streaming e transmissão, em encode offline para arquivo final ele só limita artificialmente o encoder e pode prejudicar cenas complexas

**-bufsize**

Não uso **-bufsize** pelo mesmo motivo do **-maxrate**, ele existe para controle de buffer em playback contínuo e não trouxe ganho prático nos meus testes

**-qp_i / -qp_p / -qp_b**

Não uso QP manual porque isso quebra a lógica adaptativa do AV1_QSV, quando forcei valores o encoder perdeu capacidade de reagir a mudanças de cena e a qualidade ficou irregular


**Hardware e paralelismo:**


**-low_power 1**

Não uso **-low_power** porque ele reduz a complexidade interna do encoder para economizar energia, nos testes isso piorou rate control, aumentou banding e perdeu detalhe fino, como meu foco é qualidade final esse modo só atrapalha

**-threads**

Não uso **-threads** porque o QSV ignora esse parâmetro, o paralelismo é gerenciado internamente pela GPU

**-async_depth**

Não uso **-async_depth** porque ele afeta pipeline e latência, não qualidade final, como meus encodes são offline não há benefício prático em mexer nisso


**GOP, IDR e scene change:**


**-idr_interval**

Não uso **-idr_interval** porque ele tenta impor espaçamento fixo de IDR, isso entra em conflito com GOP longo e -forced_idr, nos testes foi menos estável

**-sc_threshold**

Não uso **-sc_threshold** porque com -look_ahead_depth alto o AV1_QSV já detecta mudanças de cena de forma eficiente, forçar isso gerou decisões piores em transições suaves


**Tiles e compatibilidade:**


**-tile_cols / -tile_rows**

Não uso tiles ativos porque nas resoluções que trabalho eles não trouxeram benefício real, apenas overhead, esse recurso só começa a fazer sentido em resoluções muito maiores

**-profile / -level**

Não forço **-profile** ou **-level** manualmente porque o AV1_QSV já seleciona valores compatíveis automaticamente, forçar isso não trouxe ganho e pode até quebrar compatibilidade


**AQ (Adaptive Quantization)**


**-spatial_aq 1**

Não uso -spatial_aq porque ele tenta redistribuir bits com base em complexidade espacial do frame, na prática em anime isso tende a supervalorizar áreas com textura artificial e subvalorizar linhas finas e gradientes, nos meus testes o resultado foi mais variação de qualidade entre cenas e menos previsibilidade no tamanho final

**-temporal_aq 1**

Não uso **-temporal_aq** porque ele redistribui bits com base em variação temporal entre frames, em anime isso nem sempre funciona bem já que muitas cenas têm pouca mudança real mas exigem preservação de detalhe, nos testes esse parâmetro interferiu negativamente no rate control junto com **-look_ahead_depth** alto e **-extbrc ativo**.

**Nota sobre AQ no AV1_QSV**

No meu pipeline o controle adaptativo já é feito de forma mais eficiente através de **-look_ahead_depth** alto, **-extbrc** ativo e B-frames agressivos, adicionar **-spatial_aq** ou **-temporal_aq** acabou criando decisões redundantes ou conflitantes, o resultado prático foi perda de consistência entre episódios e menor controle sobre o teto de tamanho, ou seja, não são parâmetros errados, eles só não se encaixam no meu cenário específico de anime com limite rígido de tamanho e foco em previsibilidade visual



**Muxagem de áudio.**

**Single‑audio (faixa 0 do input):**

O vídeo já está encodeado em AV1_QSV, aqui apenas adiciono a faixa principal do input usando libopus, ajusto bitrate e VBR para manter tamanho e qualidade previsíveis, o mapeamento do vídeo é feito com -c:v copy para não reencodear, assim o processo é rápido e seguro, qualquer ajuste de idioma ou título de faixa é feito previamente no MKVToolNix.


```bash
ffmpeg \
  -i "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv" \
  -i "/run/media/malk/Downloads/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a libopus -b:a 96k -vbr constrained \
  "/run/media/malk/Downloads/output_qsv_final_q24_opus96k.mkv"
```

**Dual‑audio (faixas 0 e 1 do input):**

```bash
ffmpeg \
  -i "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv" \
  -i "/run/media/malk/Downloads/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a:0 libopus -b:a:0 80k -vbr:a:0 constrained \
    -metadata:s:a:0 title="Japonês[Malk]" \
  -map 1:a:1 -c:a:1 libopus -b:a:1 80k -vbr:a:1 constrained \
    -metadata:s:a:1 title="Português[Malk]" \
  "/run/media/malk/Downloads/output_qsv_dualaudio_q24_opus80k.mkv"
```

O **-vbr constrained** é usado para manter bitrate e tamanho previsíveis, consistente com o encode de vídeo feito anteriormente, garantindo que o arquivo final não ultrapasse limites de tamanho nem perca qualidade perceptível.

Nota: FFmpeg só consegue reconhecer idioma se a faixa já tiver metadata correta, por isso eu prefiro nomear e organizar faixas no MKVToolNix antes do mux, assim evito problemas e mantenho flexibilidade, esse método funciona tanto para single quanto para dual audio e garante consistência com o encode de vídeo feito anteriormente.

