# Croc: Legend of the Gobbos Remastered — Tradução PT-BR

Tradução para o **Português do Brasil** de *Croc: Legend of the Gobbos Remastered* (PC), com instalador automático e desinstalação limpa.

Feita por **MrGeH**.

> Projeto de fã, sem fins lucrativos e sem relação com a Argonaut Games, a Titanium Studios ou as lojas onde o jogo é vendido. Você precisa ter o jogo original.

---

## Download

Baixe o arquivo `Croc Traducao PT-BR MrGeH.zip` na página de **[Releases](../../releases)**.

## O que é traduzido

- Menu principal, pausa, fim de jogo e mensagens de salvar/apagar
- Opções de controles, gráficos, tela e áudio
- Nomes das 45 fases, dos chefes e dos segredos
- Tela de conquistas (títulos e mensagens da interface)
- Toda a **Crocipedia** (conteúdo bônus): descrições das imagens, produtos, arte conceitual, fases de teste, biografias das entrevistas e textos dos remixes
- Nomes das faixas da trilha sonora, títulos dos créditos, nomes das teclas e botões do controle

Os trocadilhos dos nomes das fases foram adaptados (por exemplo, *"lights camel action"* → *"luz, camelo, ação"*).

**Fica em inglês:** nomes próprios e títulos de jogos/filmes/revistas (seguindo as orientações dos tradutores oficiais), nomes de pessoas nos créditos, nomes e descrições das conquistas da loja, textos desenhados em imagens e os vídeos.

## Requisitos

- *Croc: Legend of the Gobbos Remastered* para PC
- Windows 10 ou 11 (não precisa instalar nada além disso)

## Instalação

1. Feche o jogo.
2. Extraia o `.zip` em qualquer pasta.
3. Execute **`Instalar Traducao PT-BR.exe`**.
4. Confira a pasta do jogo (a que tem o `Croc64.exe`). Na versão GOG o instalador encontra a pasta sozinho; caso contrário, clique em **Procurar...**.
5. Clique em **Instalar tradução**.

Não é preciso mudar o idioma do Windows nem do jogo.

## Desinstalação

Execute o instalador de novo e clique em **Desinstalar (voltar ao inglês)**. Também é possível usar "Verificar/Reparar arquivos" no GOG Galaxy ou na Steam.

## Perguntas frequentes

**Preciso mudar o idioma do Windows?**
Não. O jogo escolhe o idioma pelo Windows, mas só reconhece `en-US`, `de-DE`, `fr-FR`, `it-IT` e `es-ES`. Em qualquer outro idioma (como `pt-BR`) ele usa o inglês, então a tradução é gravada no lugar do inglês e aparece automaticamente.

**Meu Windows está em espanhol, francês, alemão ou italiano.**
O jogo continuará no idioma do Windows, pois ele tem tradução oficial para esses idiomas. Só os créditos da tradução aparecem.

**A tradução sumiu depois de uma atualização.**
Atualizações e a verificação de arquivos restauram os originais. Execute o instalador de novo.

**O antivírus reclamou do instalador.**
O executável não tem assinatura digital. O código-fonte está neste repositório, e você pode compilá-lo você mesmo (veja abaixo).

---

## Como funciona

O remaster guarda os textos da interface em um banco SQLite, `croc.db`, na tabela `LangTokens`:

```sql
CREATE TABLE LangTokens(
    id    INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    token TEXT NOT NULL,
    lang  TEXT DEFAULT NULL,  -- código ISO 639-1
    value TEXT NOT NULL,
    UNIQUE(token, lang)
)
```

O jogo busca cada texto com:

```sql
SELECT value FROM LangTokens WHERE token = ?1 AND (lang = ?2 OR lang IS NULL) ORDER BY lang DESC LIMIT 1;
```

Como o executável tem uma lista fixa de idiomas e usa o inglês para os demais, o instalador:

1. Grava cada texto traduzido como `lang='pt'` **e** por cima de `lang='en'`.
2. Grava o título do crédito da tradução também em `de`, `fr`, `es` e `it`, para ele aparecer em qualquer idioma.
3. Insere o crédito nos arquivos da interface (RmlUi):
   - `remaster\ui\frontend.rml` e `remaster\ui\frontend.rcss`: texto no canto da tela principal
   - `remaster\ui\frontend\credits-html.html`: seção nos créditos finais

Para desinstalar, o instalador apaga as linhas `pt`, devolve os textos originais em inglês (que ele carrega embutidos) e remove só os blocos que ele mesmo inseriu. Por isso a desinstalação não depende de backup e funciona mesmo depois de instalar várias vezes.

O banco é acessado pelo `winsqlite3.dll`, que já vem no Windows 10/11, e a interface do instalador usa Windows Forms (.NET Framework 4).

## Estrutura do repositório

```
├── README.md
├── LEIA-ME.txt                 # instruções que vão junto no .zip
├── Descricao NexusMods.txt     # texto da página no Nexus Mods (BBCode)
└── src/
    ├── Instalador.cs           # código do instalador (C#)
    ├── compilar.bat            # compila o instalador
    ├── traducao.tsv            # textos em português (token<TAB>texto)
    ├── ingles_original.tsv     # textos originais em inglês, usados para desinstalar
    └── exportar_traducao.py    # gera ingles_original.tsv a partir do croc.db original
```

### Formato dos arquivos `.tsv`

- `traducao.tsv`: `token<TAB>texto`
- `ingles_original.tsv`: `token<TAB>1<TAB>texto original`, ou `token<TAB>0<TAB>` quando o token não existe no jogo original (tokens novos, como os créditos da tradução)

Quebras de linha, tabulações e barras invertidas são escritas como `\n`, `\t` e `\\`.

Alguns textos contêm marcações que **precisam ser mantidas** exatamente como estão:

| Marcação | Exemplo |
|---|---|
| Botões de ação | `<actionbtn action='select'/> Selecionar` |
| Valores dinâmicos | `<data handle='keep_timer'/>`, `{{num_achieved}}` |
| Formatação C | `%s`, `%d`, `%%` |
| Espaço HTML | `&nbsp;` |

## Compilando

Não é preciso instalar o Visual Studio: o compilador C# já vem com o Windows.

```bat
src\compilar.bat
```

O instalador é gerado como `Instalar Traducao PT-BR.exe` na raiz do projeto.

### Testando pela linha de comando

```bat
"Instalar Traducao PT-BR.exe" /instalar "C:\caminho\do\jogo"
"Instalar Traducao PT-BR.exe" /desinstalar "C:\caminho\do\jogo"
```

O resultado é salvo em `traducao_ptbr_log.txt`, dentro da pasta do jogo. Teste sempre numa **cópia** da pasta do jogo.

## Contribuindo

Encontrou um erro de tradução ou um texto cortado na tela? Abra uma [issue](../../issues) com a tela onde aparece e, se possível, uma captura de tela.

Para corrigir um texto:

1. Edite a linha correspondente em `src/traducao.tsv`, mantendo as marcações da tabela acima.
2. Se adicionar um token novo, gere de novo o `ingles_original.tsv` a partir de um `croc.db` **original** (sem a tradução):
   ```bat
   python -I src\exportar_traducao.py "C:\caminho\do\jogo\croc.db" src\traducao.tsv src\ingles_original.tsv
   ```
3. Rode `src\compilar.bat` e teste numa cópia do jogo.
4. Envie um pull request.

**Nunca** envie arquivos do jogo (`croc.db`, `.rml`, `.rcss`, `.html`, imagens etc.) para o repositório.

## Créditos

- **Tradução e instalador:** MrGeH
- **Jogo:** Argonaut Games e Titanium Studios

*Croc: Legend of the Gobbos* e todas as marcas relacionadas pertencem aos seus respectivos donos. Os textos originais do jogo incluídos neste repositório servem apenas para permitir a tradução e a desinstalação.
