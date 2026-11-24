# HU Conecta

Plataforma para conectar estudantes, residentes, docentes e profissionais da saúde
do HUAC. Nesta versão **tudo acontece dentro do sistema**: o cadastro, o login, a
edição do perfil e a exclusão da conta. Os dados ficam num banco **SQL**
(SQLite por padrão; PostgreSQL, SQL Server ou MySQL trocando uma linha no `.env`).

## Como rodar

```bash
pip install -r requirements.txt
copy .env.example .env          # (Linux/Mac: cp) e edite o .env
python database.py              # cria as tabelas (só na primeira vez)
python popular_exemplo.py       # opcional: 4 pessoas fictícias, senha "teste1234"
streamlit run app_streamlit.py
```

Sem servidor de e-mail ainda? Ponha `EMAIL_MODO=console` no `.env`: os e-mails
(inclusive os códigos de confirmação) aparecem no terminal. Para enviar de verdade,
configure o bloco `SMTP_*` e confira com `python testar_email.py seu@email`.

> **Banco antigo:** esta versão tem tabelas/colunas novas (ex.: `pessoas.lattes`).
> Não reaproveite o `huconnect.db` da versão com REDCap — apague-o ou use outro
> `DATABASE_URL`.

## O que o sistema faz

- **Criar conta** (aba no login): formulário completo (dados, vínculo, papel,
  interesses e subáreas, competências, disponibilidade, senha). O cadastro só é
  concluído depois de digitar o **código de 6 dígitos enviado ao e-mail**
  (15 min, 5 tentativas, 1 pedido por minuto), então ninguém cria conta com o
  e-mail de outra pessoa. Só e-mails dos domínios de `DOMINIOS_PERMITIDOS`
  (`auth.py`) são aceitos.
- **Entrar / Esqueci minha senha**: senha guardada só como hash
  (PBKDF2-SHA256, 600 mil iterações); recuperação por código no e-mail.
- **Recomendações** por compatibilidade (papel 30%, interesses 25%, atividade,
  competências e disponibilidade 15% cada — `matching.py`), com "?" explicando
  cada critério e **resumo de interesses** em comum.
- **Avisos por e-mail**: quando alguém demonstra interesse (sem revelar contato)
  e quando o interesse é mútuo (com o contato das duas pessoas).
- **Editar meu perfil**: ao salvar, as compatibilidades da pessoa são recalculadas
  na hora (`gerar_matches.recalcular_matches_da_pessoa`); não precisa rodar nada.
- **Excluir minha conta**: apaga perfil, senha, conexões e avisos (pede a senha).
- **API (FastAPI)** opcional: `uvicorn main:app` — login, recomendações,
  interações e recuperação de senha.

## Arquivos principais

| Arquivo | Função |
|---|---|
| `app_streamlit.py` | Telas (login, cadastro, recomendações, perfil) |
| `formulario.py` | Formulário de perfil (cadastro e edição) |
| `perfil.py` | Regras de validação do formulário e conversão para o banco |
| `cadastro.py` | Cadastro com confirmação, edição e exclusão de conta |
| `recuperacao.py` | Esqueci minha senha |
| `notificacoes.py` / `email_service.py` | Like/skip e e-mails |
| `matching.py` / `gerar_matches.py` | Cálculo de compatibilidade |
| `auth.py` | Hash de senha, códigos, domínios permitidos |
| `rotulos.py` + `dados/opcoes.json` | Listas de opções (áreas, competências...) e textos |
| `models.py` / `database.py` | Tabelas e conexão SQL |
| `definir_senha.py` | Administração: definir senha pela linha de comando |

## Mudando as opções do formulário

As listas (áreas e subáreas de interesse, competências, objetivos, vínculo, curso,
instituições) estão em `dados/opcoes.json`, no formato `{campo: {código: texto}}`.
Para **acrescentar** uma opção, adicione um código novo ao campo. Evite
reutilizar ou renumerar códigos que já foram escolhidos por pessoas cadastradas,
senão os perfis antigos passam a mostrar o texto errado.

## Testes

```bash
python -m unittest discover tests
```

`tests/test_servicos.py` usa um banco temporário e e-mails simulados (não mexe nos
seus dados).

## Ainda não tem (ideias para as próximas etapas)

- Foto de perfil (o formulário antigo tinha upload).
- Opção de cada pessoa desligar os avisos por e-mail.
- Limite de tentativas de login erradas.
- Migrações de banco (Alembic) para evoluir tabelas sem apagar dados.
