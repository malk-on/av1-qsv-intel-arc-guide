# AV1 QSV Encoding Guide for Intel Arc on Linux

A complete and practical guide to encoding AV1 using Intel Arc GPUs (QSV) on Linux, with FFmpeg 7+, optimized filters, and real-world examples.

---

## 🧠 Why this guide?

This project aims to help Linux users take full advantage of Intel Arc GPUs for AV1 encoding using `av1_qsv`.  
It's based on extensive testing with FFmpeg, real use cases, and performance comparisons, all done on a modest yet modern setup.

---

## 🛠️ System Used

- **Distro:** Fedora 42 KDE  
- **Kernel:** 6.14.11-300.fc42.x86_64  
- **GPU:** Intel Arc A310  
- **CPU:** Ryzen 5 4600G  
- **FFmpeg:** 7.1.1  
- **Driver stack:** Intel Media Driver (iHD) + OneVPL  

---

## 🎥 AV1_QSV Encoding

> 📝 **Sobre os caminhos de entrada/saída:**  
> O caminho `/run/media/malk/Downloads/input.mkv` é um exemplo real.  
> Você deve ajustar conforme o local do seu arquivo, mantendo as aspas.
> global_quality deve ser ajustado individualmente.
> a entrada/saída do exemplo: "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv" tambem deve ser ajustada de acordo.

---

### ▶️ Para fontes AVC 8 bits (H.264)
```bash
ffmpeg \
  -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -color_primaries bt709 -color_trc bt709 -colorspace bt709 \
  -i "/run/media/malk/Downloads/input.mkv" \
  -vf "zscale=transfer=bt709:primaries=bt709:matrix=bt709:range=limited,format=p010le" \
  -map 0:v:0 -c:v av1_qsv \
    -global_quality 24 -preset veryslow \
    -extbrc 1 -look_ahead_depth 40 \
    -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
    -tile_cols 2 -tile_rows 1 \
    -forced_idr 1 \
  -an \
  "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv"
```



### ▶️ Para fontes AVC 10 bits (H.264)
```bash
ffmpeg \
  -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -color_primaries bt709 -color_trc bt709 -colorspace bt709 \
  -i "/run/media/malk/Downloads/input.mkv" \
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
  "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv"
```


###▶️ Para fontes HEVC 8 bits
```bash
ffmpeg -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -hwaccel qsv -hwaccel_output_format qsv -c:v hevc_qsv \
  -i "/run/media/malk/Downloads/input.mkv" \
  -vf "format=yuv420p, zscale=transfer=bt709:primaries=bt709:matrix=bt709:range=tv, format=p010le" \
  -map 0:v:0 -c:v av1_qsv \
  -global_quality 24 -preset veryslow \
  -extbrc 1 -look_ahead_depth 40 \
  -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
  -tile_cols 2 -tile_rows 1 \
  -forced_idr 1 \
  -an \
  "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv"
```

###▶️ Para fontes HEVC 10 bits
```bash
ffmpeg -init_hw_device qsv=hw:/dev/dri/renderD128 \
  -hwaccel qsv -hwaccel_output_format qsv -c:v hevc_qsv \
  -i "/run/media/malk/Downloads/input.mkv" \
  -vf "zscale=transfer=bt709:primaries=bt709:matrix=bt709:range=tv, format=p010le" \
  -map 0:v:0 -c:v av1_qsv \
  -global_quality 24 -preset veryslow \
  -extbrc 1 -look_ahead_depth 40 \
  -adaptive_i 1 -adaptive_b 1 -b_strategy 1 -bf 7 \
  -tile_cols 2 -tile_rows 1 \
  -forced_idr 1 \
  -an \
  "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv"
```


🔊 Audio Muxing with libopus


🎧 Single-audio (faixa 0 do input)
```bash
ffmpeg \
  -i "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv" \
  -i "/run/media/malk/Downloads/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a libopus -vbr off -b:a 96k \
  "/run/media/malk/Downloads/output_qsv_final_q24_opus96k.mkv"
```


🎧 Dual-audio (faixas 0 e 1 do input)
```bash
ffmpeg \
  -i "/run/media/malk/Downloads/output_av1_qsv_main10_q24.mkv" \
  -i "/run/media/malk/Downloads/input.mkv" \
  -map 0:v:0 -c:v copy \
  -map 1:a:0 -c:a:0 libopus -vbr off -b:a:0 96k -metadata:s:a:0 title="Japonês[Malk]" \
  -map 1:a:1 -c:a:1 libopus -vbr off -b:a:1 96k -metadata:s:a:1 title="Português[Malk]" \
  "/run/media/malk/Downloads/output_qsv_dualaudio_q24_opus96k.mkv"
```

🧠 Final Notes

* bf 7, adaptive_b 1, tile_cols e extbrc foram essenciais para alcançar qualidade e compressão equilibradas.

*  Arquivos mantêm compatibilidade com players modernos e excelente performance para uso local.

* Todos os testes foram feitos com decodificação por software (AVC) e QSV (HEVC), e codificação exclusivamente via av1_qsv.

📺 [Configuração recomendada do mpv no Windows](./mpv-config-windows.md)



## Por que escolhi AV1 com QSV?

Veja a explicação completa sobre minha escolha por AV1 com QSV em vez de SVT-AV1, HEVC ou AVC:

👉 [por-que-av1-qsv.md](./por-que-av1-qsv.md)


## ⚠️ Observações sobre decodificação QSV e VAAPI

- **Decodificação AVC (H.264):** Até o momento, **não é possível usar QSV ou VAAPI para decodificação de vídeos AVC (8 e 10 bits) durante o processo de encoding com av1_qsv**.  
- **Decodificação HEVC e AV1:** Para fontes HEVC (8/10 bits) e AV1, a decodificação via QSV/VAAPI funciona corretamente e pode ser usada para acelerar o processamento.  
- **Reprodução com MPV:** Apesar da limitação acima para encoding, **o VAAPI pode ser utilizado para decodificação de AVC 8 bits na reprodução via MPV**, mas AVC 10 bits com VAAPI ainda não está suportado.  
- **Recomendação prática:** Para evitar erros e garantir estabilidade no encoding AV1 via QSV, prefira usar **decodificação por software para fontes AVC**.  
- Essas descobertas foram testadas e confirmadas no ambiente Fedora 42 KDE com Intel Arc A310 e FFmpeg 7.1.1.


  






  
