# Por que escolhi AV1 com QSV em vez de SVT-AV1, HEVC ou AVC

Muita gente pergunta por que existe tanto foco em otimizar o uso do QSV, já que o encoding por software, como o SVT-AV1, ainda é tratado como referência máxima de qualidade, na prática isso ignora completamente custo computacional, tempo, consumo energético e usabilidade real do sistema, para usar SVT-AV1 em presets realmente eficientes como 4 ou 6 é necessário uma CPU de altíssimo desempenho, como Ryzen 9 7950X, Intel i9-13900K ou superiores, mesmo nesses cenários um episódio de cerca de 20 minutos pode levar de 20 a 40 minutos para encodar, usando 100% da CPU, degradando a responsividade do sistema e consumindo uma quantidade significativa de energia.

No meu caso específico, utilizando uma Intel Arc A310 com 4 GB de GDDR6 junto de um Ryzen 5 4600G, é possível encodar o mesmo conteúdo usando av1_qsv em aproximadamente 2 a 3 minutos, mantendo o sistema totalmente utilizável durante todo o processo, com uso mínimo de CPU, a qualidade visual obtida fica muito próxima do SVT-AV1 em preset 4 quando avaliada em reprodução normal, sem análises artificiais frame a frame ou ampliações irreais, esse equilíbrio entre qualidade, tempo e eficiência é o ponto central da escolha pelo QSV.

O foco no QSV não é apenas conveniência ou facilidade, é eficiência prática, a Intel entregou um encoder AV1 por hardware acessível a partir das GPUs Arc da geração Alchemist, que funciona de forma estável no Linux e no FFmpeg, permitindo que tarefas que antes exigiam estações de trabalho de alto custo sejam realizadas em setups médios, com previsibilidade, rapidez e consumo energético muito menor.

## E por que não usar SVT-AV1 por software

O SVT-AV1 continua sendo uma excelente ferramenta e uma referência técnica em cenários específicos, especialmente quando o objetivo é extrair o máximo absoluto de compressão sem restrições de tempo, no entanto em presets mais altos como 8 ou 10, apesar de serem viáveis em CPUs mais modestas, a eficiência visual cai significativamente e deixa de competir com um encode AV1 por hardware bem configurado no mesmo tempo de processamento, na prática o ganho teórico de qualidade só aparece quando o custo computacional deixa de ser uma variável, o que não reflete o uso real da maioria das pessoas.

## E por que não usar HEVC ou AVC, já que ainda são populares

A escolha por AV1 é uma decisão de longo prazo, o codec é aberto, livre de royalties e cada vez mais adotado por navegadores, plataformas de streaming, TVs e dispositivos modernos, o HEVC continua cercado por incertezas legais relacionadas a patentes, enquanto o AVC já está tecnicamente defasado em eficiência de compressão, além disso o AV1 entrega mais qualidade com menos bitrate, reduzindo tamanho de arquivo, tempo de upload, consumo de banda e necessidade de armazenamento.

## Conclusão

Com a disponibilidade de encoders AV1 por hardware como o QSV das GPUs Intel Arc, o AV1 deixou de ser lento ou experimental, hoje é uma solução madura, eficiente e plenamente viável para uso diário, mesmo em máquinas intermediárias, a escolha pelo QSV não ignora qualidade, ela prioriza eficiência real, estabilidade e resultados consistentes, alinhados com uso prático e não com benchmarks artificiais.
