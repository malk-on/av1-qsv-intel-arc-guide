Configuração do mpv no Windows (atualizada)

Eu uso o mpv no Linux faz tempo, sempre ajustando a imagem pra manter o traço limpo e natural, sem deixar nada artificial, depois de muito teste cheguei numa configuração que considero estável e focada em fidelidade visual.

A boa notícia é que dá pra usar praticamente a mesma lógica no Windows, muda apenas a forma de instalação e o local do arquivo de configuração, o comportamento visual final é o mesmo.

Como instalar o mpv no Windows

Você pode baixar uma versão já compilada do mpv direto no SourceForge, essa opção costuma ser a mais simples pra quem só quer baixar e usar.

[https://sourceforge.net/projects/mpv-player-windows/](https://sourceforge.net/projects/mpv-player-windows/)

Outra opção é instalar pelo Scoop ou Chocolatey, que são gerenciadores de pacotes e facilitam bastante as atualizações futuras.

Onde fica o arquivo de configuração

No Windows o arquivo de configuração fica em:

C:\Users\SeuUsuario\AppData\Roaming\mpv\mpv.conf

Basta criar ou abrir esse arquivo em qualquer editor de texto, Bloco de Notas, VSCode ou Notepad++, e colar a configuração abaixo.

Configuração recomendada para Windows (mpv.conf)

# Decodificação acelerada por hardware

hwdec=auto

# Renderer gpu-next usando Direct3D 11

vo=gpu-next
gpu-api=d3d11
gpu-context=auto

# Escalonadores de alta qualidade focados em preservação

scale=ewa_lanczossharp
cscale=ewa_lanczos
dscale=mitchell
tscale=oversample
sigmoid-upscaling=yes
sigmoid-slope=5

# Debanding com microgrão leve para suavizar artefatos de compressão

deband=yes
deband-iterations=2
deband-threshold=48
deband-grain=2

# Dither e framebuffer em alta precisão para evitar banding

dither-depth=auto
temporal-dither=yes
fbo-format=rgba16f

O que cada parte faz, em resumo

Hwdec=auto usa a aceleração de vídeo disponível na sua placa, Intel, AMD ou Nvidia, se notar problemas de cor, banding estranho ou incompatibilidade, vale testar hwdec=dxva2-copy ou até hwdec=no.

Vo=gpu-next com gpu-api=d3d11 força o uso do backend moderno do mpv usando Direct3D 11, que hoje é o caminho mais estável e previsível no Windows.

Scale, cscale, dscale e tscale controlam como o vídeo é redimensionado quando a resolução não é nativa da tela, a escolha aqui prioriza linhas limpas, boa definição e mínimo ringing, sem adicionar nitidez artificial.

Sigmoid-upscaling ajuda a preservar contraste perceptual durante upscale, evitando a imagem lavada comum em escalonamento linear, o slope 5 é um equilíbrio seguro para anime, podendo ser ajustado caso a fonte seja muito ruim ou muito limpa.

Deband remove faixas visíveis em gradientes causadas por compressão e baixo bit depth, o grain=2 adiciona um microgrão quase imperceptível que devolve naturalidade e evita aparência plástica.

Dither e fbo-format trabalham juntos para manter transições suaves de cor, especialmente importantes em monitores 8 bits ou conteúdos com muitos degradês.

Considerações finais

Essa configuração é pensada como set and forget, basta colar no mpv.conf e usar, sem necessidade de ajustes constantes.

Ela não altera cores, não muda o traço e não aplica nenhum tipo de aprimoramento moderno, apenas corrige limitações técnicas comuns de encode e reprodução.

O foco é preservar a intenção visual original do master, entregando uma imagem limpa, estável e natural.

Configuração equivalente para Linux (mpv.conf)

Abaixo está a configuração completa usada no Linux, mantendo exatamente a mesma filosofia visual, as diferenças existem apenas por causa do backend gráfico e do método de decodificação, não há mudança conceitual.

# Decodificação acelerada por hardware

hwdec=vaapi-copy

# Renderer gpu-next usando Vulkan

vo=gpu-next
gpu-api=vulkan
gpu-context=auto

# Escalonadores de alta qualidade focados em preservação

scale=ewa_lanczossharp
cscale=ewa_lanczos
dscale=mitchell
tscale=oversample
sigmoid-upscaling=yes
sigmoid-slope=5

# Debanding com microgrão leve

deband=yes
deband-iterations=2
deband-threshold=48
deband-grain=2

# Dither e framebuffer em alta precisão

dither-depth=auto
temporal-dither=yes
fbo-format=rgba16f

# Aspect ratio, desativado por padrão, ativar apenas se necessário

# video-aspect-override=16:9

# video-unscaled=no

Observação rápida

Em GPUs ou drivers mais antigos pode ser interessante testar fbo-format=rgba16 no lugar de rgba16f caso haja stutter ou consumo excessivo de VRAM, fora isso a configuração acima funciona como referência direta.
