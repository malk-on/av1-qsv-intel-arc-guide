# Por que escolhi AV1 com QSV em vez de SVT-AV1, HEVC ou AVC

Muita gente me pergunta por que tanto foco em otimizar o uso do QSV, já que o encoding por software, como o SVT-AV1, ainda é considerado o padrão de referência em qualidade. Mas a realidade prática é outra: pra usar SVT-AV1 com qualidade decente (como preset 4 ou 6), é necessário ter uma CPU de altíssimo desempenho, como um Ryzen 9 7950X (16c/32t), Intel i9-13900K (24 núcleos híbridos), ou até um Threadripper.

Mesmo nessas máquinas, um episódio de 20 minutos pode levar de 20 a 40 minutos pra ser encodado, usando 100% da CPU, travando o sistema e gastando muito mais energia.

No meu caso, com uma Intel Arc A310 (4GB GDDR6) e um Ryzen 5 4600G (6c/12t), consigo encodar com `av1_qsv` em apenas 2 a 3 minutos, com qualidade visual muito próxima do SVT-AV1 (preset 4, CRF 35). O uso de CPU é mínimo, e o sistema continua totalmente responsivo durante o processo.

Ou seja, o foco no QSV não é só por praticidade: é eficiência pura. A Intel entregou um encoder AV1 por hardware acessível, que funciona muito bem mesmo em setups médios. O que antes exigia uma estação high-end, hoje pode ser feito com rapidez, estabilidade e ótima qualidade.

---

## E por que não usar HEVC ou AVC, já que ainda são populares?

Simples: o futuro é o AV1.

Ele é aberto, sem royalties, e cada vez mais suportado por navegadores, plataformas, TVs e streamings como YouTube e Netflix. Ao contrário do HEVC, que tem problemas com patentes, e do AVC, que já está ultrapassado em eficiência.

Além disso, o AV1 entrega mais qualidade com menos bitrate, o que reduz o tamanho dos arquivos e facilita uploads, armazenamento e distribuição — especialmente em redes com banda limitada.

---

## Conclusão

Com encoders por hardware como o QSV da Intel Arc, o AV1 deixou de ser algo “lento” ou “experimental”. Agora é acessível, viável e altamente eficiente — até pra quem não tem uma máquina monstruosa.
