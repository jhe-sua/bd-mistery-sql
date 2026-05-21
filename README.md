## Trabalho
Esse trabalho foi feito para conclusão de uma das atividades passadas pelo professor Julio César, professor da disciplina de Banco de Dados na FGV-EMAp.

Alunos:
- Angel Yeshua
- Samuel Uchoa
- Lisânia Soares
- Davi Dressler
- Joana Fontes
- Gabriel Bittencourt

# Acesso ao site interativo de reolução do mistério

https://jhe-sua.github.io/bd-mistery-sql/

# 🔍 Mistério SQL — Operação Fantasma da Previdência

> **Polícia Federal do Brasil — DICOR/RJ — Confidencial**  
> Uso exclusivo da investigação | Divisão de Crimes Financeiros

---

## Sobre o projeto

Mistério SQL é um jogo investigativo em dois episódios onde você assume o papel do **Agente Lucas Mendonça (PF-2026-11492)**, um agente federal em seu primeiro caso de campo. O crime foi registrado em **17/05/2026 às 23h19**: alguém usou o CPF de um homem morto para fazer uma reserva no Airbnb.

Sua missão é resolver o caso escrevendo queries SQL.

---

## Sua ficha

| Campo | Dados |
|---|---|
| Nome | Lucas Mendonça |
| Matrícula | PF-2026-11492 |
| Cargo | Agente Federal — Div. de Crimes Financeiros |
| Supervisor | Delegado Marcelo Teixeira |


---

## A denúncia

Às 23h08 de 17 de maio de 2026, **Mariana Drummond Soares** recebeu uma notificação do Airbnb: alguém havia feito uma solicitação de reserva para o apartamento dela na Rua Cruz Lima, Flamengo — usando o CPF do seu pai, **falecido há dois meses**.

Ela acionou a PF imediatamente e esse caso caiu no nosso colo. Nosso objetivo inicial é confirmar que o crime realmente aconteceu. Foi requisitado à plataforma Airbnb o envio de informações do sistema como usuários e locações feitas. Mariana nos forneceu o **ID da locação: #1047**.

**Para confirmar o crime (Estelionato Majorado — Art. 171, §3 CP), nós precisamos confirmar as seguintes informações:**

1. Verificar que a locação realmente foi feita
1. Certidão de óbito de quem realizou a reserva 
2. Ao menos um saque na conta do falecido **após** a data do óbito

## Filtrar suspeitos
Depois que nós confirmamos que o crime aconteceu, precisamos verficar suspeitos com base nas informações que nos foram passadas, será que há algo suspeito no registro do usuário que fez a reserva no Airbnb? Não tem regra para identificar isso, esse é o nosso trabalho, não somos chamados de agentes por nada, vamos usar nossa experiência para prosseguir. Nós possuímos informações sobre depoimentos de pessoas próximas ao pai de Mariana, talvez seja interessante começarmos por aí.

Se conseguirmos encontrar alguma informação do registro do usuário que tenha relação com algum registros de quem prestou os depoimentos, já temos um filtro para os nossos suspeitos.

A partir de uma lista de suspeitos vamos afunilar um pouco mais, vamos verificar a situação financeira deles, será que os valores declarados em suas declarações de renda batem com os valores reais? Situações que levantam suspeitas:

1. Diferença entre renda declarada e estimada **acima de R$ 7.000** 

Esse filtro provavelmente vai nos retornar uma lista menor de suspeitos mais relevantes.

Você tem suspeitos e os dados relevantes para a investigação, conclua o trabalho agente. Boa sorte.

---

## Estrutura do banco de dados

O banco `misterio_sql` contém as seguintes tabelas:

### Episódio 1 

| Tabela | O que contém |
|---|---|
| `locacoes_airbnb` | Reservas do Airbnb — ponto de entrada: `id_locacao = 1047` |
| `usuarios_airbnb` | Cadastro dos usuários — CPF e e-mail do solicitante suspeito |
| `certificados_obito` | Certidões de óbito — confirmar se o CPF pertence a um falecido |
| `depoimentos` | Depoimentos oficiais — rastrear fragmento de e-mail com `LIKE` |
| `declaracao_renda` | Declarações de renda — filtrar diferença > R$ 7.000 |
| `contas_bancarias` | Contas bancárias — cruzar com o CPF do óbito |
| `transacoes_bancarias` | Transações — saques após `data_obito`; os `codigo_autorizador` são a pista central |
| `movimentos_suspeitos` | Registros de atividade suspeita por CPF |
| `controle_jogo` | Onde você submete a resposta do Ep. 1 |
| `tabela_criptografada_identidades` | Identidades reais por trás dos `codigo_autorizador`|

### Episódio 2

| Tabela | O que contém |
|---|---|
| `logs_criacao_autorizadores` | Registros de quem criou cada autorizador e como |
| `funcionarios_dicor` | Quadro de funcionários da DICOR — nome, matrícula, nível de acesso |
| `cadastro_empresas` | Empresas registradas — razão social, sócios, receita declarada |
| `transferencias_externas` | Transferências por autorizador para CNPJs externos |
| `mensagens_recuperadas` | Mensagens recuperadas — visíveis com credencial especial |
| `controle_jogo_ep2` | Onde você submete evidências e resposta do Ep. 2 |
| `mensagens_recuperadas` | Mensagens recuperadas|

---

## Episódio 1 — Do Airbnb ao Consultor

**8 passos**

### Passos 1–4: Do Airbnb ao Suspeito

**Passo 1 — Comece pelo Airbnb**  
Use o `id_locacao = 1047` para fazer JOIN entre `locacoes_airbnb` e `usuarios_airbnb`. Identifique o nome, CPF e o fragmento de e-mail do usuário que fez a reserva. Preste atenção no campo `senha_metodo`.

**Passo 2 — Confirme o óbito**  
Cruze o CPF encontrado com a tabela `certificados_obito`. Anote a `data_obito` — ela será o marco central de toda a investigação.

**Passo 3 — Rastreie o fragmento de e-mail**  
Use `LIKE` com o fragmento do e-mail encontrado na tabela `depoimentos` (campo `email_contato`). Atenção: mais de uma testemunha vai aparecer — o fragmento não identifica ninguém sozinho. Anote **todos** os CPFs retornados.

**Passo 4 — Filtre por inconsistência de renda**  
Cruze os CPFs com a tabela `declaracao_renda`. Quem tem `(renda_real_estimada - renda_declarada) > 7000`? Esse é o limiar do estelionato majorado. Quem ultrapassa esse valor é seu principal suspeito.

### Passos 5–8: Da Conta ao Culpado

**Passo 5 — Confirme com movimentos suspeitos pós-óbito**  
Verifique a tabela `movimentos_suspeitos` filtrando pelo CPF suspeito e `data_registro` posterior à data do óbito. O que ele fez nos dias seguintes à morte da vítima?

**Passo 6 — Saques na conta do falecido**  
Faça JOIN entre `transacoes_bancarias` e `contas_bancarias`. Filtre saques com `data_transacao` posterior à data do óbito. Anote **todos** os `codigo_autorizador` que aparecerem.

**Passo 7 — Acesse a tabela restrita e descarte os legítimos**  
Mude para o usuário `investigador_autorizado` e consulte `tabela_criptografada_identidades`. Você encontrará múltiplos autorizadores — alguns têm justificativa legal documentada. Descarte-os. Um autorizador não retornará resultado. Outro é o culpado.

**Passo 8 — Submeta o culpado principal**  
Execute o `UPDATE` na tabela `controle_jogo`.

---

## Episódio 2 — O Fantasma

**7 passos** 

O Ep. 1 encerrou com um culpado identificado — mas o autorizador **AUT-999** ficou sem dono. Alguém sacou R$ 15.000 da conta de um homem morto e não deixou rastro na tabela de identidades. Alguém de dentro.

### Passos 1–4: AUT-999 e o Rastro do Dinheiro

**Passo 1 — Investigue os logs de criação**  
Consulte `logs_criacao_autorizadores` filtrando por `AUT-999`. Anote: `metodo_insercao`, `credencial_origem`, `matricula_operador` e `ip_origem`.

**Passo 2 — Compare com os demais autorizadores**  
Liste todos os registros ordenados por `data_criacao`. Compare o `metodo_insercao` do AUT-999 com os demais. O que é diferente?

**Passo 3 — Identifique o operador pela matrícula**  
Cruze o `matricula_operador` do AUT-999 com a tabela `funcionarios_dicor`. Filtre por `nivel_acesso = 'ADMIN'` e `ativo = TRUE`. Quantos ADMINs estavam ativos em maio/2026?

**Passo 4 — Rastreie o destino do dinheiro**  
Em `transferencias_externas`, filtre por `codigo_autorizador = 'AUT-999'`. Faça JOIN com `cadastro_empresas` pelo campo `cnpj_destino`. Examine: `razao_social`, `receita_declarada`, `endereco` e — especialmente — `cpf_socio_2`.

### Passos 5–7: Mensagens, Prova e Submissão

**Passo 5 — Leia as mensagens com credencial especial**  
Consulte `mensagens_recuperadas`. O campo `remetente_cpf` agora fica visível. Cruze com `funcionarios_dicor` para confirmar a identidade do remetente.

**Passo 6 — Monte a prova completa (Query Síntese)**  
Use `UNION ALL` ligando:
- `logs_criacao_autorizadores` + `funcionarios_dicor` (quem criou o AUT-999)
- `transferencias_externas` + `cadastro_empresas` + `funcionarios_dicor` (quem recebeu o dinheiro)

O mesmo nome deve aparecer nos dois lados da prova.

**Passo 7 — Submeta o Fantasma** ⚠️ IRREVERSÍVEL  
Insira o nome do suspeito em `campo_resposta`.

<!-- > **⚠️ AVISO CRÍTICO:** Após o UPDATE, o banco é destruído. Provas que existem só dentro de um banco destruído não valem nada num tribunal. Salve tudo antes. -->

---

## Dicas gerais

- Sempre anote os resultados das queries antes de avançar — alguns dados desaparecem
- Nem todo saque pós-óbito é ilegal — verifique a justificativa antes de acusar
- A diferença de renda sozinha não basta; você precisa cruzar com outras evidências

---

<!-- ## Estrutura do repositório

```
misterio_sql/
├── README.md
├── schema.sql          # Criação das tabelas
├── seed_ep1.sql        # Dados do Episódio 1
├── seed_ep2.sql        # Dados do Episódio 2
├── roles.sql           # Criação dos usuários e permissões
└── triggers.sql        # Triggers de autodestruição -->
<!-- ```

--- -->

## Créditos

**Mistério SQL — Operação Fantasma da Previdência**    
Agente Lucas Mendonça | PF-2026-11492 | Divisão de Crimes Financeiros — DICOR/RJ

---

*Confidencial — Uso Exclusivo da Investigação | POLÍCIA FEDERAL / DICOR/RJ*
