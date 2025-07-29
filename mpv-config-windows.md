# Configuração do mpv no Windows

Faz tempo que uso o mpv no Linux, com várias configurações que ajustei depois de testar bastante pra não estragar o traço do vídeo e manter uma boa qualidade na imagem.

No Linux, meu mpv roda via Flatpak com configs personalizadas que usam Vulkan, filtros top de escalonamento e debanding, e outras coisas que deixam a imagem melhor, sem perder a naturalidade.

Mas aí pensei: será que dá pra replicar isso no Windows? Para a galera que costuma ver re-encodes lá? Porque cada player, sistema e hardware muda um pouco o visual.

Então fiz uns testes e vou contar o que funciona e o que não funciona, pra quem quiser usar mpv no Windows com configuração parecida, aproveitando o que dá.

## Como instalar o mpv no Windows

Você pode baixar o mpv já pronto no SourceForge, que tem versões atualizadas que funcionam bem no Windows:

👉 [https://sourceforge.net/projects/mpv-player-windows/](https://sourceforge.net/projects/mpv-player-windows/)

Também dá pra instalar pelo **Scoop** ou **Chocolatey**, que são gerenciadores de pacotes e facilitam atualizar depois.

## Onde fica o arquivo de configuração?

Depois de instalado, o arquivo principal pra configurar o mpv no Windows é:

C:\Users\SeuUsuario\AppData\Roaming\mpv\mpv.conf


É só abrir esse arquivo num editor de texto (Notepad, VSCode, etc) e colar as configurações.

## Configuração recomendada para Windows (mpv.conf)

```conf
vo=gpu
gpu-api=d3d11
hwdec=auto

scale=ewa_lanczossharp
cscale=ewa_lanczossharp
dscale=mitchell
tscale=oversample

deband=yes
deband-iterations=2
deband-threshold=48
deband-grain=0

dither-depth=auto
temporal-dither=yes
fbo-format=rgba16f

# video-aspect-override=16:9
# video-unscaled=no
```
Obs: as duas últimas linhas estão comentadas para manter o aspecto original (4:3 ou 16:9 conforme o vídeo). No Linux eu costumo forçar pra esticar 4:3 em 16:9, mas no Windows recomendo deixar como está.

Sobre a aceleração por hardware

* Intel (incluindo Arc): hwdec=auto funciona bem

* Nvidia: hwdec=auto ativa NVDEC/NVENC

* AMD: idem

* Sem aceleração: use hwdec=no para forçar o uso da CPU

Considerações finais

Fiz esses ajustes testando com os mesmos vídeos que uso no Linux, rodando mpv no Windows, pra ter uma base visual parecida.

Não fica 100% igual (por causa das diferenças de drivers e APIs), mas a imagem não perde qualidade e nem fica artificial.



🧩 A configuração que uso no mpv é o que fecha o pacote do meu encode.

Como o encode tem limitações naturais (por causa do tamanho do arquivo e do uso do QSV), rolar uns artefatos é normal.

Mas com os filtros de escala, debanding e dithering que coloquei no mpv, a imagem fica mais suave, o traço é preservado e os defeitos visíveis somem ou ficam bem menos incômodos.

No fim das contas, a config do mpv ajuda a entregar uma experiência visual bem melhor, compensando as limitações técnicas do encode e deixando tudo redondinho pra quem assiste.

