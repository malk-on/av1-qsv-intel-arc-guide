🎬 AV1 QSV Encoding Guide for Intel Arc on Linux:

Um guia completo e prático pra quem quer tirar o máximo do AV1 usando GPUs Intel Arc no Linux, via Quick Sync Video (QSV). Baseado em testes reais, filtros otimizados e exemplos de uso do dia a dia, usando FFmpeg 7+.

🧠 Por que montar esse guia?

A ideia aqui é ajudar outros usuários Linux a aproveitar a placa Intel Arc de forma eficiente no encode AV1, já que a documentação oficial costuma ser confusa ou incompleta. Todo esse material saiu de testes práticos em um setup real, pra facilitar a vida de quem quer qualidade e performance sem esquentar a cabeça.

🛠️ Sistema de testes:

Distro: Fedora 42 KDE

Kernel: 6.14.11

GPU: Intel Arc A310

CPU: Ryzen 5 4600G

FFmpeg: 7.1.1

Driver stack: Intel Media Driver (iHD) + OneVPL


🎥 Encoding AV1 via QSV (Intel Arc):

📝 Sobre os caminhos de entrada/saída:

Os caminhos que aparecem abaixo (/run/media/malk/Downloads/input.mkv etc) são exemplos reais que eu uso.

Ajuste pro local do seu arquivo, mantendo as aspas.

O global_quality também vale a pena ajustar conforme sua fonte e necessidade.

▶️ Fontes AVC 8-bit (H.264):
```bash
ffmpeg \
  -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -color_primaries bt709 -color_trc bt709 -colorspace bt709 \
  -i "/caminho/do/seu/input.mkv" \
  -vf "zscale=transfer=bt709:primaries=bt709:matrix=bt709:range=limited,format=p010le" \
  -map 0:v:0 -c:v av1_qsv \
    -global_quality 24 -preset veryslow \
    -extbrc 1 -look_ahead_depth 40 \
    -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
    -tile_cols 2 -tile_rows 1 \
    -forced_idr 1 \
  -an \
  "/caminho/do/seu/output_av1_qsv_main10_q24.mkv"
```

▶️ Fontes AVC 10-bit (H.264):
```bash
ffmpeg \
  -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -color_primaries bt709 -color_trc bt709 -colorspace bt709 \
  -i "/caminho/do/seu/input.mkv" \
  -vf "format=yuv420p,\
zscale=transfer=bt709:primaries=bt709:matrix=bt709:range=limited,\
format=p010le" \
  -map 0:v:0 -c:v av1_qsv \
    -global_quality 24 -preset veryslow \
    -extbrc 1 -look_ahead_depth 40 \
    -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
    -tile_cols 2 -tile_rows 1 \
    -forced_idr 1 \
  -an \
  "/caminho/do/seu/output_av1_qsv_main10_q24.mkv"
```

▶️ Fontes HEVC 8-bit:
```bash
ffmpeg \
  -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -filter_hw_device hw \
  -hwaccel qsv -hwaccel_output_format qsv -c:v hevc_qsv \
  -i "/caminho/do/seu/input.mkv" \
  -vf "hwdownload,format=yuv420p, \
       zscale=transfer=bt709:primaries=bt709:matrix=bt709:range=tv, \
       format=p010le,hwupload=extra_hw_frames=64,format=qsv" \
  -map 0:v:0 -c:v av1_qsv \
    -global_quality 24 -preset veryslow \
    -extbrc 1 -look_ahead_depth 40 \
    -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
    -tile_cols 2 -tile_rows 1 \
    -forced_idr 1 \
  -an \
  "/caminho/do/seu/output_av1_qsv_main10_q24.mkv"
```

▶️ Fontes HEVC 10-bit:
```bash
ffmpeg \
  -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -hwaccel qsv -hwaccel_output_format qsv -c:v hevc_qsv \
  -i "/caminho/do/seu/input.mkv" \
  -map 0:v:0 -c:v av1_qsv \
    -global_quality 24 -preset veryslow \
    -extbrc 1 -look_ahead_depth 40 \
    -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
    -tile_cols 2 -tile_rows 1 \
    -forced_idr 1 \
  -an \
  "/caminho/do/seu/output_av1_qsv_main10_q24.mkv"
```

🎧 Mux de Áudio com libopus:

Faixa única (primeiro áudio do input):
```bash
ffmpeg \
  -i "/caminho/do/seu/output_av1_qsv_main10_q24.mkv" \
  -i "/caminho/do/seu/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a libopus -b:a 96k \
  "/caminho/do/seu/output_final_q24_opus96k.mkv"
```

Dual áudio (faixas 0 e 1 do input):
```bash
ffmpeg \
  -i "/caminho/do/seu/output_av1_qsv_main10_q24.mkv" \
  -i "/caminho/do/seu/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a:0 libopus -b:a:0 96k -metadata:s:a:0 title="Japonês[Malk]" \
  -map 1:a:1 -c:a:1 libopus -b:a:1 96k -metadata:s:a:1 title="Português[Malk]" \
  "/caminho/do/seu/output_dualaudio_q24_opus96k.mkv"
```

🧠 Notas finais:

Os parâmetros bf 7, adaptive_b 1, tile_cols e extbrc fizeram muita diferença na qualidade e no tamanho final.

Todos os testes foram feitos com decodificação por software no caso do AVC, e via QSV no caso do HEVC.

Arquivos gerados rodam suave em players modernos.

👉 Se quiser, também recomendo dar uma olhada no meu preset pro MPV no Windows:

📺 [Configuração recomendada do mpv no Windows](mpv-config-windows.md)


⚠️ Observações sobre decodificação:

No Linux com placas Intel Arc, rolou o seguinte nos testes:

### 🧩 Tabela de suporte à decodificação no Linux com Intel Arc

| Formato de entrada | QSV Funciona? | VAAPI Funciona? | Observações |
|--------------------|---------------|------------------|-------------|
| AVC 8-bit          | ❌ Instável   | ⚠️ Apenas reprodução (MPV) | Use `-hwaccel none` |
| AVC 10-bit         | ❌ Não suportado | ❌ Não suportado | Use `-hwaccel none` |
| HEVC 8-bit         | ✅ Sim        | ✅ Sim           | Estável e rápido |
| HEVC 10-bit        | ✅ Sim        | ✅ Sim           | Ideal para pipelines QSV |
| AV1 8/10-bit       | ❌ Crasha    | ❌ Crasha        | Use `-hwaccel none` |

Nota: Esses testes foram no Fedora 42 KDE com a Arc A310. No Windows o comportamento pode ser diferente, principalmente com driver Intel oficial.

🤔 Por que AV1 via QSV?

Se quiser entender direitinho o porquê da minha escolha de AV1 com QSV (ao invés de SVT-AV1, HEVC, AVC…), tem um texto separado só pra isso:

👉 [Por que escolhi AV1 com QSV](por-que-av1-qsv.md)






