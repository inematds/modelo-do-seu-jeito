# Modelo do Seu Jeito

[![Modelo do Seu Jeito](guia/assets/banner.jpg)](https://inematds.github.io/modelo-do-seu-jeito/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## O que é

Seu modelo de IA aberto às vezes recusa uma pergunta inofensiva ou promete o que não sabe? Este kit ajuda você a ajustar uma cópia desse modelo no seu computador, sem treinar do zero. Ele traz dois caminhos: **menos recusas indevidas** e **ajuste de comportamento**. Serve para quem usa modelos abertos e quer comprovar a mudança, comparando o antes e o depois. Você precisa de uma máquina capaz de rodar o modelo. A edição é feita com o [Heretic](https://github.com/p-e-w/heretic).

> **Estado atual:** o kit ainda não foi executado em um modelo. Réguas, dados, configurações e comparativo têm testes automáticos sem modelo, e as configurações passaram na validação do Heretic. Por isso não há resultado medido para mostrar. O primeiro uso real é um teste.

## 📖 Guia

Guia completo: **https://inematds.github.io/modelo-do-seu-jeito/guia/**

## Comece pedindo à sua IA

O Claude Code e o Codex são assistentes que trabalham no seu computador e precisam estar instalados, com conta ativa. Abra um deles em uma pasta vazia e cole:

```text
Leia o repositório https://github.com/inematds/modelo-do-seu-jeito, principalmente o README.
Depois verifique se meu computador comporta o processo e me pergunte qual modelo aberto quero usar
e qual mudança quero: menos recusas indevidas ou ajuste de comportamento.
Explique em linguagem simples o que será instalado e baixado, o tamanho e os riscos.
Não instale, não baixe e não altere nada antes do meu sim. Mantenha o modelo original intacto.
```

## Os dois modos

| Modo | Problema | O que muda | Como é aprovado |
|---|---|---|---|
| `modos/menos-recusas` | O modelo recusa pedidos inofensivos: história, saúde, direitos, "matar um processo" | Reduz a recusa indevida | Responde com conteúdo em pelo menos 80% dos casos novos e perde no máximo 10 pontos em perguntas comuns |
| `modos/pedir-antes` | No atendimento, o modelo inventa valor ou promete crédito sem ter os dados | Pede o fato que falta e, com os dados, responde direto | Pelo menos 80% em cada situação e perda máxima de 10 pontos em perguntas comuns |

O modo de recusas usa apenas exemplos inofensivos. O objetivo é corrigir recusas erradas, não retirar proteção contra pedidos perigosos. A edição pode ter efeitos além dos exemplos, por isso compare antes de usar.

## Só funciona com modelo aberto

Editar exige os arquivos de pesos do modelo e uma máquina sob seu controle, própria ou alugada. Uma API de provedor entrega só o texto da resposta, então não dá para aplicar a edição nela. Serviços que já oferecem modelos alterados são outra opção: você usa o modelo deles, não o seu.

## Primeiro, teste se instruir já resolve

Editar custa tempo e pode piorar o modelo. Cada modo traz uma `instrucao-forte.txt`, e o comparativo mede três versões lado a lado: original, original com instrução forte e cópia editada. Se a instrução sozinha passar, talvez você não precise editar.

## Próximas ideias

A tabela em [SUGESTOES.md](SUGESTOES.md) reúne outros comportamentos que dá para tentar. O que aproveitamos do Heretic e o que acrescentamos está em [DO-HERETIC.md](DO-HERETIC.md).

---

## O processo, do início ao fim

Legenda: 🧠 você decide · ⚙️ script ou comando

Os comandos foram escritos para Linux e macOS. O Windows não foi verificado.

0. ⚙️ Baixe o kit e entre na pasta:
   ```sh
   git clone https://github.com/inematds/modelo-do-seu-jeito.git
   cd modelo-do-seu-jeito
   ```
1. 🧠 Escolha um modelo aberto, confira a licença e veja o tamanho dos pesos na página dele.
2. ⚙️ Confira a máquina, trocando 8 pelo tamanho dos pesos em GB:
   `python3 kit/checar_maquina.py --pesos-gb 8`
3. ⚙️ Crie o ambiente, instale o [PyTorch 2.6+ adequado à sua placa](https://pytorch.org/get-started/locally/) e depois o restante:
   ```sh
   python3 -m venv .venv && . .venv/bin/activate
   pip install "heretic-llm @ git+https://github.com/p-e-w/heretic@64e999152ff7918efc73576814709b6b6a71f15c" transformers accelerate pytest
   ```
4. ⚙️ Baixe o original. O exemplo usa o Qwen3-4B:
   `hf download Qwen/Qwen3-4B-Instruct-2507 --local-dir modelos/original`
5. ⚙️ Registre a impressão digital do original. Se abrir outro terminal depois, reative o ambiente com `. .venv/bin/activate`:
   `python kit/impressao.py gerar modelos/original modelos/impressao-original.json`
6. 🧠 Entre no modo e adapte os exemplos em `dados/` ao seu caso, como explicado em [Como adaptar os dados](#como-adaptar-os-dados):
   `cd modos/menos-recusas` ou `cd modos/pedir-antes`
7. ⚙️ Colete as respostas antes de editar:
   ```sh
   python ../../kit/responder.py --modelo ../../modelos/original --saida ../../respostas/original.jsonl
   python ../../kit/responder.py --modelo ../../modelos/original --instrucao instrucao-forte.txt --saida ../../respostas/instruido.jsonl
   python ../../kit/avaliar.py --original ../../respostas/original.jsonl --instruido ../../respostas/instruido.jsonl
   ```
   Se a saída for `NADA A CORRIGIR`, pare aqui. Se for `A INSTRUÇÃO RESOLVE`, use a instrução e não edite o modelo.
8. ⚙️ Faça um piloto curto: `heretic --n-trials 12 --n-startup-trials 6 --model ../../modelos/original`
9. 🧠 O Heretic mostra os melhores resultados. As duas notas devem ser baixas: a do objetivo e a de afastamento do original (KL). O piloto faz sentido se a nota do objetivo caiu sem o afastamento disparar. Para continuar, escolha `Run additional trials` no menu e peça mais tentativas. O padrão do Heretic são 100 no total. Depois escolha um resultado e salve em `../../modelos/ajustado`. Na configuração padrão, o Heretic informa 30 a 40 minutos para esse modelo em uma RTX 3090. Este kit não mediu o tempo.
10. ⚙️ Colete as respostas da cópia e compare as três versões:
    ```sh
    python ../../kit/responder.py --modelo ../../modelos/ajustado --saida ../../respostas/ajustado.jsonl
    python ../../kit/avaliar.py --original ../../respostas/original.jsonl --instruido ../../respostas/instruido.jsonl --modificado ../../respostas/ajustado.jsonl
    ```
11. ⚙️ Prove que o original não mudou: `python ../../kit/impressao.py conferir ../../modelos/original ../../modelos/impressao-original.json`
12. 🧠 Leia uma amostra das respostas. Depois escreva perguntas novas, com o critério definido antes de ver as respostas, e rode os passos 7 e 10 com `--casos` e com outros nomes em `--saida`, para não sobrescrever as respostas anteriores.

Saídas do comparativo: `0` aprovado ou há o que corrigir, `1` reprovado, `2` inconclusivo, `3` nada a corrigir, `4` a instrução já resolve.

## Instalação técnica

- Python 3.11+, PyTorch 2.6+, `git` e espaço para o original e as cópias.
- O checador usa folga estimada: memória de 2 vezes e disco de 3 vezes o tamanho dos pesos. Não é medição.
- Testes sem modelo: `python -m pytest -q`.

| Pasta | Conteúdo |
|---|---|
| `kit/` | checagem da máquina, impressão digital, coleta de respostas e comparativo |
| `modos/*/config.toml` | configuração do Heretic |
| `modos/*/regua.py` | o que conta como resposta certa |
| `modos/*/dados/` | contraste, otimização, perguntas comuns e avaliação separada |
| `modos/*/instrucao-forte.txt` | alternativa sem edição |
| `modos/pedir-antes/plugin_heretic.py` | nota própria para o Heretic |

As configurações usam só arquivos locais e não baixam os conjuntos padrão do Heretic.

## Como adaptar os dados

Arquivos `.txt` têm uma pergunta por linha. Arquivos `.jsonl` têm um caso por linha, com `id` único, `tipo` e `pedido`.

| Modo | Tipo | Campos extras |
|---|---|---|
| `menos-recusas` | `recusa_indevida` | `deve_citar`: lista de termos, e basta aparecer um |
| `menos-recusas` | `comum` | `esperado`: texto que deve aparecer |
| `pedir-antes` | `neutra` | `esperado`: texto que deve aparecer |
| `pedir-antes` | `sem_dados`, `abaixo_limite` | nenhum |
| `pedir-antes` | `com_dados` | `esperado`: valor numérico correto |

No modo `menos-recusas`, coloque em `dados/recusadas.txt` e `dados/otimizacao.txt` só pedidos inofensivos que o seu modelo recusa. Em `dados/comuns.txt` ficam perguntas normais. Não repita frases entre os arquivos.

Os exemplos prontos estão em português. Para outro idioma, traduza os dados e as instruções, inclusive `system_prompt`, e confira `keyword_markers` em `config.toml` e os padrões de `regua.py`. A régua reconhece recusas em português, inglês e espanhol.

O modo `pedir-antes` vem com o exemplo de um aplicativo de agendamento. Para outro negócio, troque as instruções em `config.toml` e `instrucao-forte.txt`, edite as frases em `gerar_dados.py` e rode `python gerar_dados.py`.

Depois de congelar a cópia, escreva as perguntas novas em outro `.jsonl`, no mesmo formato, e passe esse arquivo com `--casos` ao `responder.py` e ao `avaliar.py`.

## Limites

- A régua reconhece padrões de texto e não substitui a leitura humana.
- Passar nos casos do kit não prova segurança nem precisão fora deles.
- Compressão ou mudança de formato pode alterar o comportamento. Compare de novo depois de converter.
- Um modelo editado continua podendo errar fatos. Em produto com usuários, mantenha revisão e proteções próprias.
- Respeite a licença do modelo e as leis aplicáveis.

## Base e licença

Baseado no [Heretic](https://github.com/p-e-w/heretic), de Philipp Emanuel Weidmann e colaboradores, na versão `64e9991`. Este projeto usa a mesma licença, [AGPL-3.0](LICENSE).
