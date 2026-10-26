# Declaro Meu Tempo

Aplicativo simples, sem servidor, para que cada pessoa de uma equipe declare os
horários e períodos em que prevê estar disponível no trabalho — "de 01/09 a 31/12,
de segunda a sexta, das 9h às 15h". Leitores veem as declarações das suas unidades;
administradores veem todas, em tabelas e gráficos. É uma **previsão de
disponibilidade**, não um ponto eletrônico: nada é conferido contra o que aconteceu,
e nada vira métrica de desempenho. Pensado principalmente para o serviço público
brasileiro.

- **Sem infraestrutura:** abre-se `code/index.html` direto da pasta, no Chrome ou no
  Edge. Os dados ficam numa pasta de rede do Windows, acessada pelo navegador.
- **Documentação:** `code/LEIA-ME.txt` explica cada decisão; `project_design.md` é a
  referência estruturada (esquemas, papéis, decisões, roteiro).
- **Testes:** `node testes/rodar.mjs` (Node 22+ e Chrome ou Edge, sem pacotes npm);
  execuções registradas em `testes/REGISTRO.md`.

Até a v0.8 o projeto se chamou "Declaração de disponibilidade" e viveu no repositório
[`ErnyBSB/declare-time`](https://github.com/ErnyBSB/declare-time), hoje arquivado.
As issues e os PRs citados nos documentos com números até #32 estão lá.

Licença: [Unlicense](LICENSE) (domínio público).
