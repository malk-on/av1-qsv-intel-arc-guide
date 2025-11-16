🎬 AV1 QSV — Guia prático para Intel Arc no Linux

Um guia direto ao ponto pra extrair o máximo do AV1 usando Intel Arc (QSV) no Linux. Baseado em testes reais, exemplos de uso do dia a dia e filtros ajustados para FFmpeg 7+.
A ideia é: facilitar a vida de quem quer boa qualidade e desempenho sem ficar perdido na documentação oficial.

Sistema de testes

* Distro: Fedora 42 KDE

* Kernel: 6.14.11

* GPU: Intel Arc A310

* CPU: Ryzen 5 4600G

* FFmpeg: 7.1.1

* Driver stack: Intel Media Driver (iHD) + OneVPL

Caminhos de exemplo:

Os caminhos que aparecem abaixo (/run/media/malk/Downloads/input.mkv etc.) são exemplos reais que eu uso — mantenha as aspas e ajuste pro seu local.

O -global_quality merece teste: ajuste conforme sua fonte e objetivo (qualidade vs tamanho).


* Importante: decodificação por QSV para AVC é instável no Linux com Intel Arc.
Por isso, sempre use decodificação por software em fontes H.264, mesmo as 8-bit.

▶️ Fontes AVC 8-bit (H.264):
```bash
ffmpeg \
 -init_hw_device qsv=hw:/dev/dri/renderD128 \
 -filter_hw_device hw \
 -i "/run/media/malk/Downloads/input.mkv" \
 -map 0:v:0 \
 -vf "hwupload=extra_hw_frames=64,format=qsv,scale_qsv=format=p010" \
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

▶️ Fontes AVC 10-bit (H.264):
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

▶️ Fontes HEVC 8-bit:
```bash
ffmpeg \
 -init_hw_device qsv=hw:/dev/dri/renderD128 \
 -filter_hw_device hw \
 -hwaccel qsv -hwaccel_output_format qsv -c:v hevc_qsv \
 -i "/run/media/malk/Downloads/input.mkv" \
 -map 0:v:0 -c:v av1_qsv \
   -preset veryslow \
   -global_quality 24 \
   -look_ahead_depth 100 -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 8 \
   -extbrc 1 -g 300 -forced_idr 1 \
   -tile_cols 0 -tile_rows 0 \
   -pix_fmt yuv420p10le \
 -an \
 "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv"
```

▶️ Fontes HEVC 10-bit:
```bash
ffmpeg \
 -init_hw_device qsv=hw:/dev/dri/renderD128 \
 -filter_hw_device hw \
 -hwaccel qsv -hwaccel_output_format qsv -c:v hevc_qsv \
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

🎧 Mux de Áudio com libopus:

Faixa única (primeiro áudio do input):
```bash
ffmpeg \
  -i "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv"\
  -i "/run/media/malk/Downloads/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a libopus -vbr off -b:a 96k \
  "/run/media/malk/Downloads/output_qsv_final_q24_opus96k.mkv"
```

Dual áudio (faixas 0 e 1 do input):
```bash
ffmpeg \
  -i "/run/media/malk/Downloads/output_av1_qsv_ultramax_q24.mkv" \
  -i "/run/media/malk/Downloads/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a:0 libopus -vbr off -b:a:0 80k -metadata:s:a:0 title="Japonês[Malk]" \
  -map 1:a:1 -c:a:1 libopus -vbr off -b:a:1 80k -metadata:s:a:1 title="Português[Malk]" \
  "/run/media/malk/Downloads/output_qsv_dualaudio_q24_opus96k.mkv"
```

🧠 Notas finais:

* Para fontes AVC (H.264): prefira decodificação por software — QSV para decodificação AVC pode falhar em várias fontes. Use -i input.mkv sem -hwaccel qsv quando a origem for AVC.

* Para HEVC/AV1: decodificação por QSV costuma funcionar bem em Arc. Testei isso no meu setup (Fedora + Arc A310).

 * Se a sua fonte for 8→10 bits (ou vice-versa), cuidado com scale_qsv=format=p010 / -pix_fmt yuv420p10le — preserve o formato correto pra evitar banding/desbalanceamento de cores.

* -global_quality é o principal controle de qualidade: experimente na prática (valores típicos que eu testo: ~18 = mais qualidade / pesado, até ~30 = mais compacto). Ajuste pra sua fonte.

* -look_ahead_depth, -adaptive_i, -adaptive_b, -b_strategy, -bf e -extbrc são parâmetros que otimizei pra Arc — podem ser reduzidos se você precisar de encode mais rápido.

* -g 300 e -forced_idr 1 funcionam bem pra controle de GOP em animes/filmes, mas ajuste conforme sua timeline de capítulos/cenas.

* tile_cols / tile_rows ficam em 0 aqui (auto). Em conteúdo muito grande (4K+), testar tiles pode acelerar/ajustar performance.

* Containers: uso MKV por estabilidade com múltiplos áudios/legendas; MP4 pode dar problemas com alguns codec metadata.

* Sempre verifique sua versão do FFmpeg e drivers (OneVPL/iHD). Pequenas versões mudam comportamento do av1_qsv.

* Teste em pequenos cortes primeiro (10–30s) antes de rodar o arquivo inteiro — economiza tempo ao ajustar -global_quality.

* QSV aceita apenas YUV420. Se sua fonte for 4:2:2 ou 4:4:4, faça a conversão antes:
-vf format=yuv420p10le (ou yuv420p para 8-bit).


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
| AV1 8/10-bit       | ❌ Instável no pipeline QSV    | ❌ Instável no pipeline QSV | Use `-hwaccel none` |


*Resumo: usando Arc no Linux, o mais seguro e estável é sempre decodificar por software. Use QSV apenas na parte de encode.

Nota: Esses testes foram no Fedora 43 KDE com a Arc A310. No Windows o comportamento pode ser diferente, principalmente com driver Intel oficial.

🤔 Por que AV1 via QSV?

Se quiser entender direitinho o porquê da minha escolha de AV1 com QSV (ao invés de SVT-AV1, HEVC, AVC…), tem um texto separado só pra isso:

👉 [Por que escolhi AV1 com QSV](por-que-av1-qsv.md)

* Nota Final: Todas as informações abaixo foram testadas na prática no Fedora 43 KDE, Fmpeg 7.1.1 e GPU Intel Arc A310. Muitos desses comportamentos não estão, documentados oficialmente, mas foram verificados de forma consistente em, dezenas de encodes.







