# Fundamentos do Passo 4 — Criando o Leitor/Escritor JSON: `JsonHandler.java`

Nesta etapa, construímos a camada de persistência do projeto. A classe `JsonHandler` é responsável por **salvar** a lista de tarefas em disco (em formato JSON) e **carregar** essas tarefas de volta para a memória — sem usar nenhuma biblioteca externa.

Cada engrenagem desta classe será desmontada e testada de forma isolada, exatamente como fizemos no Passo 2. Ao final, você saberá exatamente o que cada linha faz e por quê ela existe.

---

## Índice

1. [Estrutura da Classe e a Constante `CAMINHO_ARQUIVO`](#1-estrutura-da-classe-e-a-constante-caminho_arquivo)
2. [Método `saveTasks` — Salvando Tarefas no Arquivo JSON](#2-método-savetasks--salvando-tarefas-no-arquivo-json)
3. [Método `loadTasks` — Carregando Tarefas do Arquivo JSON](#3-método-loadtasks--carregando-tarefas-do-arquivo-json)
4. [Método privado `parsearTarefa` — Convertendo JSON em Objeto Task](#4-método-privado-parsearTarefa--convertendo-json-em-objeto-task)
5. [Método privado `extrairValor` — O Motor do Parsing com Regex](#5-método-privado-extrairvalor--o-motor-do-parsing-com-regex)
6. [Teste de Integração Final — O Ciclo Completo](#6-teste-de-integração-final--o-ciclo-completo)

---

## 1. Estrutura da Classe e a Constante `CAMINHO_ARQUIVO`

Antes de qualquer método, a classe declara uma constante que define onde o arquivo JSON será lido e gravado.

```java
package tasktracker;

import java.io.*;                          // IOException — classe base de erros de I/O
import java.nio.charset.StandardCharsets;  // StandardCharsets.UTF_8 — garante codificação universal
import java.nio.file.*;                    // Files, Paths — API moderna de manipulação de arquivos
import java.util.*;                        // List, ArrayList — coleções Java
import java.util.regex.*;                  // Pattern, Matcher — motor de expressões regulares

public class JsonHandler {

    // private:  Só acessível dentro desta própria classe
    // static:   Pertence à classe, não a uma instância — não precisamos fazer "new JsonHandler()"
    // final:    Constante — o valor não pode ser alterado depois de definido
    // String:   O tipo; neste caso, um caminho/nome de arquivo em texto
    // "tasks.json": O arquivo será criado/lido na pasta onde o programa for executado
    private static final String CAMINHO_ARQUIVO = "tasks.json";
}
```

### Por que `static` em tudo?

A classe `JsonHandler` não precisa de estado interno. Ela é uma coleção de **utilitários de disco** — você a usa chamando diretamente seus métodos, sem instanciar objetos:

```java
// ✅ Uso correto: método estático, sem "new"
JsonHandler.saveTasks(tarefas);
List<Task> lista = JsonHandler.loadTasks();

// ❌ Uso desnecessário: instanciar para nada
JsonHandler jh = new JsonHandler();
jh.saveTasks(tarefas);  // funcionaria, mas é confuso e sem sentido
```

### Anatomia da Declaração da Constante

```text
[Visibilidade]  ──> private
[Escopo]        ──> static   (pertence à classe, não a instâncias)
[Mutabilidade]  ──> final    (valor fixo após inicialização)
[Tipo]          ──> String
[Nome]          ──> CAMINHO_ARQUIVO   (maiúsculas por convenção de constantes)
[Valor]         ──> "tasks.json"
```

### 1.1. Testando a Constante: Onde o arquivo será criado?

#### Classe `JsonHandler.java` (módulo mínimo)

```java
package tasktracker;

import java.nio.file.Paths;  // Paths.get() — converte String em objeto de caminho do sistema

public class JsonHandler {

    // A constante que define o nome/caminho do arquivo de persistência
    // "tasks.json" sem caminho absoluto significa: pasta atual de execução do programa
    private static final String CAMINHO_ARQUIVO = "tasks.json";

    // Método auxiliar público apenas para este teste — expõe onde o arquivo seria criado
    public static String getCaminhoAbsoluto() {
        // Paths.get(String): Constrói um objeto Path a partir de um texto
        // .toAbsolutePath(): Resolve o caminho relativo para o caminho completo no sistema de arquivos
        // .toString(): Converte o objeto Path de volta para String legível
        return Paths.get(CAMINHO_ARQUIVO).toAbsolutePath().toString();
    }
}
```

#### Classe `Main.java` (visualização do resultado)

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {

        System.out.println("=== TESTANDO LOCALIZAÇÃO DO ARQUIVO JSON ===\n");

        // Chama o método estático diretamente pelo nome da classe — sem "new JsonHandler()"
        // Isso é possível porque getCaminhoAbsoluto() foi declarado com "static"
        String caminho = JsonHandler.getCaminhoAbsoluto();

        System.out.println("O arquivo tasks.json seria criado em:");
        System.out.println(caminho);

        // Mostra apenas o nome do arquivo, sem o caminho completo
        // lastIndexOf(File.separator): Encontra a posição do último separador de pasta (/ ou \)
        // substring(pos + 1): Extrai somente o nome do arquivo após o último separador
        String nomeArquivo = caminho.substring(
            caminho.lastIndexOf(java.io.File.separator) + 1
        );
        System.out.println("\nNome do arquivo: " + nomeArquivo);

        System.out.println("\n==========================================");
    }
}
```

### 1.2. Saída do Terminal (Output Esperado)

```text
=== TESTANDO LOCALIZAÇÃO DO ARQUIVO JSON ===

O arquivo tasks.json seria criado em:
/home/usuario/projetos/task-tracker/tasks.json

Nome do arquivo: tasks.json

==========================================
Process finished with exit code 0
```

> 💡 **Nota:** O caminho absoluto mostrado será diferente no seu computador — ele reflete a pasta raiz do seu projeto no IntelliJ. No Windows, os separadores serão `\` (ex: `C:\Users\usuario\projetos\task-tracker\tasks.json`).

---

## 2. Método `saveTasks` — Salvando Tarefas no Arquivo JSON

Este é o método de **escrita**. Ele recebe uma lista de objetos `Task` em memória e transforma tudo em um arquivo `.json` válido em disco.

```java
// Recebe uma lista de tarefas (pode ser vazia — nesse caso grava "[\n]")
public static void saveTasks(List<Task> tarefas) {

    // StringBuilder: Classe especializada para construir Strings longas peça por peça.
    // É muito mais eficiente que concatenar com "+", que cria um novo objeto String a cada operação.
    StringBuilder sb = new StringBuilder();

    // Inicia o conteúdo do arquivo com o colchete de abertura do array JSON
    // "\n" é o caractere de nova linha (line feed) — coloca cada objeto na sua própria linha
    sb.append("[\n");

    // Itera sobre a lista pelo índice (não pelo for-each) porque precisamos saber
    // se o elemento atual é o ÚLTIMO — para não colocar vírgula depois dele
    for (int i = 0; i < tarefas.size(); i++) {

        // tarefas.get(i): Acessa o elemento na posição i da lista (começa em 0)
        // .toJson(): Chama o método de serialização que escrevemos na classe Task —
        //            retorna a representação JSON deste objeto (o bloco { ... })
        sb.append(tarefas.get(i).toJson());

        // Regra do JSON: elementos separados por vírgula, mas o último não leva vírgula
        // Se i é menor que o índice do último elemento (tamanho - 1), adiciona vírgula + nova linha
        if (i < tarefas.size() - 1) {
            sb.append(",\n");
        }
    }

    // Fecha o array JSON com nova linha antes do colchete (estética padrão)
    sb.append("\n]");

    try {
        // Files.write(): Método estático da API NIO do Java para gravar bytes em arquivo
        // Paths.get(CAMINHO_ARQUIVO): Converte o nome do arquivo em um objeto Path do sistema
        // sb.toString(): Obtém o conteúdo completo acumulado no StringBuilder como uma String
        // .getBytes(StandardCharsets.UTF_8): Converte a String para bytes usando codificação UTF-8
        //   UTF-8 é essencial para garantir que caracteres como ã, é, ç sejam gravados corretamente
        //   sem depender da configuração regional do sistema operacional
        // Se o arquivo não existir: Files.write() o cria automaticamente
        // Se o arquivo já existir: Files.write() SOBRESCREVE o conteúdo anterior completamente
        Files.write(
                Paths.get(CAMINHO_ARQUIVO),
                sb.toString().getBytes(StandardCharsets.UTF_8)
        );

    } catch (IOException e) {
        // IOException: Exceção checada lançada quando uma operação de I/O falha
        // Motivos comuns: disco cheio, sem permissão de escrita, caminho inválido
        // System.err: Stream de erro padrão — imprime na cor vermelha no terminal do IntelliJ
        // Optamos por logar o erro e continuar (não travar o programa) — comportamento tolerante a falhas
        System.err.println("Erro ao salvar o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
    }
}
```

### 2.1. Anatomia do `StringBuilder` em Ação

```text
Estado do StringBuilder conforme o loop executa (2 tarefas):

Após sb.append("[\n"):
┌─────────┐
│ [       │
│         │
└─────────┘

Após sb.append(tarefas.get(0).toJson()):
┌─────────────────────────────┐
│ [                           │
│   {                         │
│     "id": 1,                │
│     "description": "...",   │
│     ...                     │
│   }                         │
└─────────────────────────────┘

Após sb.append(",\n")  [porque i=0 < tamanho-1=1]:
┌─────────────────────────────┐
│ [                           │
│   { ... },                  │    ← vírgula adicionada
└─────────────────────────────┘

Após sb.append(tarefas.get(1).toJson()):
┌─────────────────────────────┐
│ [                           │
│   { ... },                  │
│   { ... }                   │    ← sem vírgula (é o último)
└─────────────────────────────┘

Após sb.append("\n]"):
┌─────────────────────────────┐
│ [                           │
│   { ... },                  │
│   { ... }                   │
│ ]                           │
└─────────────────────────────┘
```

### 2.2. Código de Teste Isolado

#### Classe `JsonHandler.java` (módulo de escrita)

```java
package tasktracker;

import java.io.IOException;                // Exceção de operações de entrada/saída (leitura/escrita)
import java.nio.charset.StandardCharsets;  // Conjunto de codificações de texto (UTF-8, ISO-8859-1...)
import java.nio.file.Files;                // Utilitário estático com métodos de leitura e escrita de arquivos
import java.nio.file.Paths;                // Converte Strings em objetos Path (caminho do sistema de arquivos)
import java.util.List;                     // Interface que representa uma lista ordenada de objetos

public class JsonHandler {

    // Constante com o nome do arquivo onde as tarefas serão gravadas
    // Por ser apenas o nome sem pasta, o arquivo será criado na raiz do projeto no IntelliJ
    private static final String CAMINHO_ARQUIVO = "tasks.json";

    // [MÉTODO DO ITEM 2]: Grava a lista completa de tarefas no arquivo JSON
    // Parâmetro: List<Task> — uma lista tipada, aceita somente objetos do tipo Task
    public static void saveTasks(List<Task> tarefas) {

        // StringBuilder: Buffer mutável de caracteres — mais eficiente que String + String
        // Cada chamada a .append() adiciona conteúdo ao final do buffer sem criar novos objetos
        StringBuilder sb = new StringBuilder();

        // Inicia o JSON com o colchete de abertura do array
        sb.append("[\n");

        // Loop pelo índice para ter controle sobre se é o último elemento
        for (int i = 0; i < tarefas.size(); i++) {

            // .get(i): Recupera o elemento da lista na posição i
            // .toJson(): Método da classe Task — serializa o objeto para texto JSON formatado
            sb.append(tarefas.get(i).toJson());

            // Separador de elementos: vírgula apenas entre elementos (não após o último)
            // tarefas.size() - 1 é o índice do último elemento
            if (i < tarefas.size() - 1) {
                sb.append(",\n");  // adiciona vírgula e quebra de linha entre elementos
            }
        }

        // Fecha o array JSON
        sb.append("\n]");

        try {
            // Files.write(): Cria ou sobrescreve o arquivo com o conteúdo fornecido em bytes
            // Paths.get(): Transforma o nome do arquivo em um objeto Path que a JVM entende
            // .toString(): Obtém o conteúdo do StringBuilder como uma String normal
            // .getBytes(UTF_8): Converte a String para sequência de bytes em codificação UTF-8
            Files.write(
                    Paths.get(CAMINHO_ARQUIVO),          // onde gravar
                    sb.toString().getBytes(StandardCharsets.UTF_8)  // o quê gravar (em bytes)
            );

        } catch (IOException e) {
            // Captura qualquer falha de disco ou permissão e exibe no canal de erros
            System.err.println("Erro ao salvar o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }
    }
}
```

#### Classe `Main.java` (visualização do `saveTasks`)

```java
package tasktracker;

import java.io.IOException;       // Para poder ler o arquivo depois de salvo
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;       // Implementação de List baseada em array redimensionável
import java.util.List;

public class Main {
    public static void main(String[] args) throws IOException {

        System.out.println("=== TESTANDO saveTasks() ===\n");

        // ── Preparando os dados de teste ─────────────────────────────────────────

        // Criamos objetos Task manualmente para simular a situação real de uso
        // Construtor (int id, String description): define status="todo" e timestamps automáticos
        Task t1 = new Task(1, "Comprar leite");
        Task t2 = new Task(2, "Estudar Java");
        Task t3 = new Task(3, "Fazer exercícios");

        // Alteramos o status de algumas tarefas para testar a variedade no JSON gerado
        // setStatus(): setter que também atualiza o campo updatedAt automaticamente
        t2.setStatus("in-progress");
        t3.setStatus("done");

        // ArrayList: Implementação de List que permite adicionar/remover elementos dinamicamente
        // List<Task>: O tipo genérico <Task> garante que só objetos Task podem entrar na lista
        List<Task> tarefas = new ArrayList<>();
        tarefas.add(t1);  // .add(): Adiciona o elemento ao final da lista
        tarefas.add(t2);
        tarefas.add(t3);

        System.out.println("Tarefas em memória antes de salvar:");
        // for-each: itera sobre cada elemento da lista sem se preocupar com índices
        for (Task t : tarefas) {
            // System.out.println(objeto): Chama implicitamente o método toString() da Task
            System.out.println("  " + t);
        }

        System.out.println("\n--- Chamando JsonHandler.saveTasks() ---\n");

        // Chama o método estático — grava o arquivo tasks.json na raiz do projeto
        JsonHandler.saveTasks(tarefas);

        // ── Verificação: lemos o arquivo com Files.readString para ver o que foi gravado ──

        // Files.readString(): Lê todo o conteúdo de um arquivo como String de uma vez
        // Paths.get(): Localiza o arquivo pelo caminho
        // StandardCharsets.UTF_8: Garante decodificação correta dos caracteres especiais
        String conteudoGravado = new String(
            Files.readAllBytes(Paths.get("tasks.json")),
            StandardCharsets.UTF_8
        );

        System.out.println("Conteúdo do arquivo tasks.json gerado:");
        System.out.println("─────────────────────────────────────────");
        System.out.println(conteudoGravado);
        System.out.println("─────────────────────────────────────────");

        System.out.println("\n✅ Arquivo gravado com sucesso!");
        System.out.println("   Verifique o arquivo 'tasks.json' na raiz do seu projeto.");
        System.out.println("\n==========================================");
    }
}
```

### 2.3. Saída do Terminal (Output Esperado)

```text
=== TESTANDO saveTasks() ===

Tarefas em memória antes de salvar:
  [1] Comprar leite                              | Status: todo         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:00
  [2] Estudar Java                               | Status: in-progress  | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01
  [3] Fazer exercícios                           | Status: done         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01

--- Chamando JsonHandler.saveTasks() ---

Conteúdo do arquivo tasks.json gerado:
─────────────────────────────────────────
[
  {
    "id": 1,
    "description": "Comprar leite",
    "status": "todo",
    "createdAt": "2026-05-29 10:00:00",
    "updatedAt": "2026-05-29 10:00:00"
  },
  {
    "id": 2,
    "description": "Estudar Java",
    "status": "in-progress",
    "createdAt": "2026-05-29 10:00:00",
    "updatedAt": "2026-05-29 10:00:01"
  },
  {
    "id": 3,
    "description": "Fazer exercícios",
    "status": "done",
    "createdAt": "2026-05-29 10:00:00",
    "updatedAt": "2026-05-29 10:00:01"
  }
]
─────────────────────────────────────────

✅ Arquivo gravado com sucesso!
   Verifique o arquivo 'tasks.json' na raiz do seu projeto.

==========================================
Process finished with exit code 0
```

> 💡 **Inspecione o arquivo no IntelliJ:** Após executar este teste, o arquivo `tasks.json` aparecerá no painel **Project** (à esquerda), na raiz do projeto. Clique nele para ver o conteúdo exatamente como seria visto por qualquer editor de texto ou visualizador JSON.

---

## 3. Método `loadTasks` — Carregando Tarefas do Arquivo JSON

Este é o método de **leitura**. Ele abre o arquivo `tasks.json`, identifica cada bloco JSON `{ ... }` usando expressões regulares e reconstrói os objetos `Task` em memória.

```java
public static List<Task> loadTasks() {

    // Cria a lista vazia que será preenchida e retornada
    // ArrayList: implementação de List que cresce dinamicamente conforme elementos são adicionados
    List<Task> tarefas = new ArrayList<>();

    // File: Classe do java.io que representa um arquivo ou diretório no sistema de arquivos
    // Usada aqui apenas para verificar se o arquivo já existe — sem abri-lo ainda
    File arquivo = new File(CAMINHO_ARQUIVO);

    // Se o arquivo não existir, é a primeira execução do programa
    // Retornar lista vazia (não null!) é a decisão certa — evita NullPointerException nos chamadores
    if (!arquivo.exists()) {
        return tarefas;  // lista vazia, comportamento normal
    }

    try {
        // Files.readAllBytes(): Lê TODOS os bytes do arquivo de uma vez e os retorna como byte[]
        // Paths.get(String): Cria um objeto Path a partir do nome/caminho do arquivo
        // new String(bytes, charset): Converte o array de bytes em texto usando a codificação UTF-8
        // StandardCharsets.UTF_8: Codificação segura para caracteres acentuados (ã, é, ç, etc.)
        String conteudo = new String(
                Files.readAllBytes(Paths.get(CAMINHO_ARQUIVO)),
                StandardCharsets.UTF_8
        );

        /*
         * Pattern.compile(): Compila a expressão regular em um objeto reutilizável e eficiente
         *
         * A regex: \{[^{}]+\}
         *
         *   \{       → chave de abertura literal { (o \ é necessário porque { tem significado em regex)
         *   [^{}]+   → um ou mais (+) caracteres que NÃO sejam ( ^ ) nem { nem }
         *              isso evita capturar chaves aninhadas acidentalmente
         *   \}       → chave de fechamento literal }
         *
         * Em Java, cada \ da regex precisa ser escrito como \\ na string Java (é um escape duplo):
         *   regex:   \{[^{}]+\}
         *   Java:    "\\{[^{}]+\\}"
         *
         * Pattern.DOTALL: Flag que faz o ponto (.) do regex também casar com \n (quebra de linha)
         *   Necessário porque cada objeto JSON ocupa MÚLTIPLAS linhas no arquivo
         *   Sem DOTALL, o ponto não atravessa linhas e a captura falharia
         */
        Pattern padraoBlocoJson = Pattern.compile("\\{[^{}]+\\}", Pattern.DOTALL);

        // Matcher: Objeto que aplica o Pattern contra um texto específico
        // .matcher(conteudo): Prepara o motor de busca para varrer o conteúdo do arquivo
        Matcher matcher = padraoBlocoJson.matcher(conteudo);

        // matcher.find(): Avança para a próxima ocorrência do padrão no texto
        // Retorna true enquanto houver mais blocos { ... } para encontrar
        while (matcher.find()) {

            // matcher.group(): Retorna o texto que casou com o padrão na última chamada de find()
            // Exemplo: "{\n    \"id\": 1,\n    \"description\": \"Comprar leite\",\n    ...}"
            String blocoJson = matcher.group();

            // parsearTarefa(): Método privado (analisado na Seção 4) que converte o bloco JSON
            // em um objeto Task — pode retornar null se o bloco estiver malformado
            Task tarefa = parsearTarefa(blocoJson);

            // Só adicionamos tarefas válidas à lista (proteção contra corrupção de arquivo)
            if (tarefa != null) {
                tarefas.add(tarefa);
            }
        }

    } catch (IOException e) {
        // Captura falhas de leitura (arquivo bloqueado por outro processo, disco com erro, etc.)
        System.err.println("Erro ao ler o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
    }

    // Retorna a lista populada (ou vazia se o arquivo estiver vazio ou ocorreu erro)
    return tarefas;
}
```

### 3.1. Anatomia do Fluxo de Leitura

```text
                           ┌──────────────────────────────┐
                           │  tasks.json (em disco)        │
                           │  [                            │
                           │    { "id": 1, ... },          │
                           │    { "id": 2, ... }           │
                           │  ]                            │
                           └──────────────┬───────────────┘
                                          │
                              Files.readAllBytes()
                                          │
                                          ▼
                           ┌──────────────────────────────┐
                           │  conteudo (String na memória) │
                           │  "[\n  {\n    \"id\": 1..."   │
                           └──────────────┬───────────────┘
                                          │
                          Pattern.compile("\\{[^{}]+\\}")
                          + Matcher.find() em loop
                                          │
                              ┌───────────┴──────────┐
                              │                      │
                              ▼                      ▼
                    blocoJson #1             blocoJson #2
                    "{ "id": 1... }"        "{ "id": 2... }"
                              │                      │
                      parsearTarefa()        parsearTarefa()
                              │                      │
                              ▼                      ▼
                           Task(1,...)           Task(2,...)
                              │                      │
                              └──────────┬───────────┘
                                         ▼
                              List<Task> [t1, t2]  ← retornada
```

### 3.2. Código de Teste Isolado

Este teste carrega o arquivo **gerado pelo teste anterior** (Seção 2). **Execute os testes em ordem** — primeiro o `saveTasks`, depois este.

#### Classe `JsonHandler.java` (módulo de leitura — inclui `parsearTarefa` e `extrairValor` simplificados)

> Para este teste isolado, incluiremos versões funcionais de `parsearTarefa` e `extrairValor` — eles serão detalhados nas próximas seções.

```java
package tasktracker;

import java.io.File;                       // Representa arquivo/diretório — usamos para checar existência
import java.io.IOException;                // Exceção de operações de I/O (leitura, escrita em disco)
import java.nio.charset.StandardCharsets;  // Constantes de codificação de texto (UTF-8)
import java.nio.file.Files;                // API NIO — métodos estáticos de manipulação de arquivos
import java.nio.file.Paths;                // Converte String em objeto Path entendível pela JVM
import java.util.ArrayList;               // Implementação concreta de List (array redimensionável)
import java.util.List;                    // Interface de coleção ordenada
import java.util.regex.Matcher;           // Aplica um Pattern em um texto e localiza correspondências
import java.util.regex.Pattern;           // Representa uma expressão regular compilada

public class JsonHandler {

    // Caminho do arquivo de persistência — relativo à pasta de execução do projeto
    private static final String CAMINHO_ARQUIVO = "tasks.json";

    // [MÉTODO DO ITEM 3]: Carrega todas as tarefas salvas no arquivo JSON
    // Retorno: List<Task> — sempre uma lista (nunca null); pode estar vazia se arquivo não existe
    public static List<Task> loadTasks() {

        // Inicia a lista que será preenchida com as tarefas lidas do disco
        List<Task> tarefas = new ArrayList<>();

        // File(String path): Objeto que representa o arquivo — ainda não o abre/lê
        // .exists(): Verifica no sistema de arquivos se o arquivo existe (retorna boolean)
        File arquivo = new File(CAMINHO_ARQUIVO);

        // Primeira execução do programa: arquivo ainda não foi criado — comportamento normal
        if (!arquivo.exists()) {
            return tarefas;  // retorna lista vazia, não null — seguro para qualquer chamador
        }

        try {
            // Lê todos os bytes do arquivo e os converte em String usando codificação UTF-8
            // Files.readAllBytes(): Lê o arquivo inteiro de uma vez — bom para arquivos pequenos
            // new String(byte[], Charset): Converte bytes → texto interpretando com o charset informado
            String conteudo = new String(
                    Files.readAllBytes(Paths.get(CAMINHO_ARQUIVO)),
                    StandardCharsets.UTF_8
            );

            // Compila a regex que identifica cada bloco JSON { ... }
            // "\\{[^{}]+\\}": Em Java, \\ representa uma barra simples \  (escape duplo)
            //   A regex resultante é: \{[^{}]+\}
            //   \{    → chave de abertura literal
            //   [^{}]+→ um ou mais caracteres que NÃO sejam { ou }
            //   \}    → chave de fechamento literal
            // Pattern.DOTALL: O ponto (.) passa a casar também com quebras de linha (\n)
            //   Sem este flag, os blocos JSON multi-linha não seriam capturados
            Pattern padraoBlocoJson = Pattern.compile("\\{[^{}]+\\}", Pattern.DOTALL);

            // Cria o Matcher: objeto que percorre o texto procurando o padrão
            Matcher matcher = padraoBlocoJson.matcher(conteudo);

            // find(): Avança para a próxima correspondência — retorna false quando não há mais
            while (matcher.find()) {
                // group(): Retorna o trecho do texto que correspondeu ao padrão
                String blocoJson = matcher.group();

                // Converte o bloco JSON em objeto Task (método interno desta classe)
                Task tarefa = parsearTarefa(blocoJson);

                // Só insere na lista se a conversão foi bem-sucedida (proteção defensiva)
                if (tarefa != null) {
                    tarefas.add(tarefa);
                }
            }

        } catch (IOException e) {
            System.err.println("Erro ao ler o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }

        return tarefas;
    }

    // Métodos auxiliares necessários para o loadTasks funcionar (detalhados nas Seções 4 e 5)
    private static Task parsearTarefa(String json) {
        try {
            int id           = Integer.parseInt(extrairValor(json, "id"));
            String descricao = extrairValor(json, "description");
            String status    = extrairValor(json, "status");
            String criadoEm  = extrairValor(json, "createdAt");
            String atualizadoEm = extrairValor(json, "updatedAt");
            return new Task(id, descricao, status, criadoEm, atualizadoEm);
        } catch (Exception e) {
            System.err.println("Erro ao interpretar tarefa do JSON: " + e.getMessage());
            return null;
        }
    }

    private static String extrairValor(String json, String chave) {
        Pattern padraoString = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
        );
        Matcher m = padraoString.matcher(json);
        if (m.find()) {
            return m.group(1)
                    .replace("\\\"", "\"")
                    .replace("\\\\", "\\")
                    .replace("\\n", "\n")
                    .replace("\\r", "\r");
        }
        Pattern padraoNumero = Pattern.compile("\"" + chave + "\"\\s*:\\s*(\\d+)");
        m = padraoNumero.matcher(json);
        if (m.find()) return m.group(1);
        return "";
    }
}
```

#### Classe `Main.java` (visualização do `loadTasks`)

```java
package tasktracker;

import java.util.List;

public class Main {
    public static void main(String[] args) {

        System.out.println("=== TESTANDO loadTasks() ===\n");
        System.out.println("Lendo o arquivo 'tasks.json' gerado pelo teste anterior...\n");

        // Chama loadTasks() — lê o arquivo e reconstrói os objetos Task
        // Se o arquivo não existir, retorna lista vazia (não lança exceção)
        List<Task> tarefas = JsonHandler.loadTasks();

        // Verifica se a leitura retornou alguma tarefa
        if (tarefas.isEmpty()) {
            // .isEmpty(): Retorna true se a lista não tiver nenhum elemento
            System.out.println("⚠️  Nenhuma tarefa encontrada!");
            System.out.println("   Execute primeiro o teste do saveTasks() para criar o arquivo.");
            return;  // Encerra o método main sem continuar
        }

        // .size(): Retorna o número de elementos na lista
        System.out.println("✅ " + tarefas.size() + " tarefa(s) carregada(s) do arquivo:\n");

        // Itera sobre cada Task reconstituída e exibe seus dados individuais
        for (Task t : tarefas) {
            System.out.println("─── Tarefa carregada ───────────────────────────────");
            // Os getters abaixo provam que os campos foram corretamente desserializados
            System.out.println("  ID         : " + t.getId());
            System.out.println("  Descrição  : " + t.getDescription());
            System.out.println("  Status     : " + t.getStatus());
            System.out.println("  Criado em  : " + t.getCreatedAt());
            System.out.println("  Atualizado : " + t.getUpdatedAt());
            System.out.println("  toString() : " + t);  // toString() formata a linha completa
            System.out.println();
        }

        System.out.println("==========================================");
        System.out.println("Ciclo completo: disco → String → regex → Task ✅");
    }
}
```

### 3.3. Saída do Terminal (Output Esperado)

```text
=== TESTANDO loadTasks() ===

Lendo o arquivo 'tasks.json' gerado pelo teste anterior...

✅ 3 tarefa(s) carregada(s) do arquivo:

─── Tarefa carregada ───────────────────────────────
  ID         : 1
  Descrição  : Comprar leite
  Status     : todo
  Criado em  : 2026-05-29 10:00:00
  Atualizado : 2026-05-29 10:00:00
  toString() : [1] Comprar leite                              | Status: todo         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:00

─── Tarefa carregada ───────────────────────────────
  ID         : 2
  Descrição  : Estudar Java
  Status     : in-progress
  Criado em  : 2026-05-29 10:00:00
  Atualizado : 2026-05-29 10:00:01
  toString() : [2] Estudar Java                               | Status: in-progress  | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01

─── Tarefa carregada ───────────────────────────────
  ID         : 3
  Descrição  : Fazer exercícios
  Status     : done
  Criado em  : 2026-05-29 10:00:00
  Atualizado : 2026-05-29 10:00:01
  toString() : [3] Fazer exercícios                           | Status: done         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01

==========================================
Ciclo completo: disco → String → regex → Task ✅
Process finished with exit code 0
```

---

## 4. Método privado `parsearTarefa` — Convertendo JSON em Objeto Task

Este método recebe um **fragmento JSON** (um único bloco `{ ... }`) e o converte em um objeto `Task`. Ele é chamado pelo `loadTasks` para cada bloco encontrado.

```java
// private: Só acessível dentro da própria classe JsonHandler — é um detalhe de implementação interno
// static:  Chamado por loadTasks() (que também é static) sem precisar de instância
// Task:    Pode retornar um objeto Task válido OU null (em caso de erro de parsing)
private static Task parsearTarefa(String json) {
    try {
        // extrairValor(): Método auxiliar (Seção 5) que aplica regex no fragmento JSON
        // para encontrar e retornar o valor de um campo específico como String

        // Extrai o valor do campo "id" — vem como String "1", precisamos converter para int
        int id = Integer.parseInt(extrairValor(json, "id"));
        //        └── Integer.parseInt(): Converte String numérica para o tipo primitivo int
        //            Lança NumberFormatException se o valor não for um número válido

        // Os outros campos são Strings — não precisam de conversão de tipo
        String descricao    = extrairValor(json, "description");
        String status       = extrairValor(json, "status");
        String criadoEm     = extrairValor(json, "createdAt");
        String atualizadoEm = extrairValor(json, "updatedAt");

        // Usa o construtor de 5 parâmetros da classe Task (o construtor para tarefas EXISTENTES)
        // Este construtor não gera timestamps novos — restaura os valores exatamente como estavam
        return new Task(id, descricao, status, criadoEm, atualizadoEm);

    } catch (NumberFormatException e) {
        // Ocorre quando extrairValor(json, "id") retorna algo que não é um número
        // Ex: se o arquivo JSON foi corrompido e o campo id ficou vazio ou com texto
        System.err.println("Erro: ID inválido no JSON — " + e.getMessage());
        return null;  // null sinaliza falha — o chamador (loadTasks) ignora este bloco

    } catch (Exception e) {
        // Captura qualquer outro erro inesperado durante a conversão
        System.err.println("Erro ao interpretar tarefa do JSON: " + e.getMessage());
        return null;
    }
}
```

### 4.1. Por que retornar `null` em vez de lançar uma exceção?

```text
Design decision — Tolerância a falhas:

Opção A — Lançar exceção:
  parsearTarefa() lança exceção
        ↓
  loadTasks() para de ler todas as tarefas
        ↓
  TaskManager recebe lista vazia
        ↓
  Uma tarefa corrompida apaga TODAS as outras ❌

Opção B — Retornar null (escolhida):
  parsearTarefa() retorna null
        ↓
  loadTasks() ignora apenas essa tarefa (if tarefa != null)
        ↓
  As outras tarefas são carregadas normalmente
        ↓
  Uma tarefa corrompida não prejudica as demais ✅
```

### 4.2. Código de Teste Isolado

#### Classe `JsonHandler.java` (módulo `parsearTarefa`)

```java
package tasktracker;

import java.util.regex.Matcher;  // Aplica o Pattern contra uma String e localiza correspondências
import java.util.regex.Pattern;  // Representa uma expressão regular compilada e pronta para uso

public class JsonHandler {

    // [MÉTODO DO ITEM 4]: Converte um fragmento JSON em um objeto Task
    // Acesso privado — detalhe interno de implementação, não exposto para outros pacotes
    // Retorna null em caso de falha para isolar erros de parsing
    private static Task parsearTarefa(String json) {
        try {
            // Cada chamada a extrairValor() varre o fragmento JSON com uma regex diferente
            // buscando o par "chave": valor correspondente

            // "id" é um campo numérico no JSON — extrairValor retorna "1" (String)
            // Integer.parseInt() converte "1" → 1 (int primitivo)
            int id = Integer.parseInt(extrairValor(json, "id"));

            // Os demais campos são strings no JSON — extrairValor retorna o valor já desescapado
            String descricao    = extrairValor(json, "description");
            String status       = extrairValor(json, "status");
            String criadoEm     = extrairValor(json, "createdAt");
            String atualizadoEm = extrairValor(json, "updatedAt");

            // Task(id, desc, status, createdAt, updatedAt): Construtor para tarefas EXISTENTES
            // Diferente de Task(id, desc) que cria tarefas novas com timestamps automáticos
            return new Task(id, descricao, status, criadoEm, atualizadoEm);

        } catch (NumberFormatException e) {
            // Campo "id" com valor não-numérico (arquivo corrompido ou editado manualmente com erro)
            System.err.println("Erro: ID inválido no JSON — " + e.getMessage());
            return null;  // sinaliza falha ao chamador sem derrubar o resto da leitura

        } catch (Exception e) {
            // Qualquer outro erro de conversão inesperado
            System.err.println("Erro ao interpretar tarefa do JSON: " + e.getMessage());
            return null;
        }
    }

    // Versão pública do parsearTarefa para poder chamar diretamente nos testes
    // Em produção, este método seria privado — aqui é public apenas para fins didáticos
    public static Task parsearTarefaPublico(String json) {
        return parsearTarefa(json);
    }

    // Incluído aqui para o teste funcionar (detalhado na Seção 5)
    private static String extrairValor(String json, String chave) {
        Pattern padraoString = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
        );
        Matcher m = padraoString.matcher(json);
        if (m.find()) {
            return m.group(1)
                    .replace("\\\"", "\"")
                    .replace("\\\\", "\\")
                    .replace("\\n", "\n")
                    .replace("\\r", "\r");
        }
        Pattern padraoNumero = Pattern.compile("\"" + chave + "\"\\s*:\\s*(\\d+)");
        m = padraoNumero.matcher(json);
        if (m.find()) return m.group(1);
        return "";
    }
}
```

#### Classe `Main.java` (visualização do `parsearTarefa`)

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {

        System.out.println("=== TESTANDO parsearTarefa() ===\n");

        // ── Caso 1: JSON válido ───────────────────────────────────────────────────
        // Simula o bloco exato que o loadTasks() extrairia do arquivo
        String blocoValido =
            "{\n" +
            "    \"id\": 42,\n" +
            "    \"description\": \"Aprender Regex em Java\",\n" +
            "    \"status\": \"in-progress\",\n" +
            "    \"createdAt\": \"2026-05-29 08:00:00\",\n" +
            "    \"updatedAt\": \"2026-05-29 09:30:00\"\n" +
            "  }";

        System.out.println("JSON de entrada (Caso 1 — válido):");
        System.out.println(blocoValido);
        System.out.println();

        // parsearTarefaPublico(): Wrapper público do método privado — apenas para este teste
        Task tarefaValida = JsonHandler.parsearTarefaPublico(blocoValido);

        if (tarefaValida != null) {
            System.out.println("✅ Task reconstruída com sucesso:");
            System.out.println("   ID         : " + tarefaValida.getId());
            System.out.println("   Descrição  : " + tarefaValida.getDescription());
            System.out.println("   Status     : " + tarefaValida.getStatus());
            System.out.println("   Criado em  : " + tarefaValida.getCreatedAt());
            System.out.println("   Atualizado : " + tarefaValida.getUpdatedAt());
        } else {
            System.out.println("❌ Falha ao parsear (retornou null)");
        }

        System.out.println();

        // ── Caso 2: JSON com descrição contendo aspas escapadas ──────────────────
        String blocoComAspas =
            "{\n" +
            "    \"id\": 7,\n" +
            "    \"description\": \"Ler o livro \\\"Clean Code\\\"\",\n" +
            "    \"status\": \"todo\",\n" +
            "    \"createdAt\": \"2026-05-29 08:00:00\",\n" +
            "    \"updatedAt\": \"2026-05-29 08:00:00\"\n" +
            "  }";

        System.out.println("─────────────────────────────────────────");
        System.out.println("JSON de entrada (Caso 2 — aspas escapadas):");
        System.out.println(blocoComAspas);
        System.out.println();

        Task tarefaComAspas = JsonHandler.parsearTarefaPublico(blocoComAspas);
        if (tarefaComAspas != null) {
            System.out.println("✅ Descrição desescapada corretamente:");
            System.out.println("   Descrição  : " + tarefaComAspas.getDescription());
            // Esperado: Ler o livro "Clean Code"  (sem as barras invertidas)
        }

        System.out.println();

        // ── Caso 3: JSON corrompido (id inválido) ─────────────────────────────────
        String blocoCorreto =
            "{\n" +
            "    \"id\": ABC,\n" +  // id inválido — não é número
            "    \"description\": \"Tarefa corrompida\",\n" +
            "    \"status\": \"todo\",\n" +
            "    \"createdAt\": \"2026-05-29 08:00:00\",\n" +
            "    \"updatedAt\": \"2026-05-29 08:00:00\"\n" +
            "  }";

        System.out.println("─────────────────────────────────────────");
        System.out.println("JSON de entrada (Caso 3 — id corrompido):");
        System.out.println(blocoCorreto);
        System.out.println();

        Task tarefaCorrepta = JsonHandler.parsearTarefaPublico(blocoCorreto);
        if (tarefaCorrepta == null) {
            System.out.println("✅ Retornou null — comportamento esperado para JSON inválido");
            System.out.println("   (O erro foi impresso no System.err acima)");
        }

        System.out.println("\n==========================================");
    }
}
```

### 4.3. Saída do Terminal (Output Esperado)

```text
=== TESTANDO parsearTarefa() ===

JSON de entrada (Caso 1 — válido):
{
    "id": 42,
    "description": "Aprender Regex em Java",
    "status": "in-progress",
    "createdAt": "2026-05-29 08:00:00",
    "updatedAt": "2026-05-29 09:30:00"
  }

✅ Task reconstruída com sucesso:
   ID         : 42
   Descrição  : Aprender Regex em Java
   Status     : in-progress
   Criado em  : 2026-05-29 08:00:00
   Atualizado : 2026-05-29 09:30:00

─────────────────────────────────────────
JSON de entrada (Caso 2 — aspas escapadas):
{
    "id": 7,
    "description": "Ler o livro \"Clean Code\"",
    "status": "todo",
    "createdAt": "2026-05-29 08:00:00",
    "updatedAt": "2026-05-29 08:00:00"
  }

✅ Descrição desescapada corretamente:
   Descrição  : Ler o livro "Clean Code"

─────────────────────────────────────────
JSON de entrada (Caso 3 — id corrompido):
{
    "id": ABC,
    ...
  }

Erro: ID inválido no JSON — For input string: ""
✅ Retornou null — comportamento esperado para JSON inválido
   (O erro foi impresso no System.err acima)

==========================================
Process finished with exit code 0
```

---

## 5. Método privado `extrairValor` — O Motor do Parsing com Regex

Este é o coração do `JsonHandler`. Dado um fragmento JSON e o nome de um campo, ele usa **expressões regulares** para localizar e extrair o valor desse campo. Suporta dois tipos: strings e números.

```java
// json:  O bloco JSON completo de uma tarefa (texto bruto do arquivo)
// chave: O nome do campo a extrair — ex: "description", "id", "status"
// Retorno: O valor do campo como String — ou "" se não encontrado
private static String extrairValor(String json, String chave) {

    // ═══════════════════════════════════════════════════════════════════════════
    // CASO 1 — Campo do tipo String  (ex: "description": "Comprar leite")
    // ═══════════════════════════════════════════════════════════════════════════

    /*
     * Construção da regex passo a passo:
     *
     * "chave"                 → o nome do campo exato entre aspas duplas
     *   Em Java: "\"" + chave + "\""
     *
     * \s*:\s*                 → dois-pontos com zero ou mais espaços em volta
     *   Em Java: "\\s*:\\s*"
     *   \s = qualquer espaço em branco (espaço, tab, \n)
     *   *  = zero ou mais repetições
     *
     * "                       → abre a string JSON com aspas duplas literais
     *   Em Java: "\""
     *
     * (                       → INÍCIO do grupo de captura (o que queremos extrair)
     *
     *   (?:                   → grupo NÃO-capturante (agrupa sem numerar como grupo)
     *     [^"\\]              → qualquer caractere exceto " e \
     *     |                   → OU
     *     \\.                 → uma barra invertida seguida de QUALQUER caractere
     *                           isso captura sequências de escape: \", \\, \n, \r
     *   )*                    → repete zero ou mais vezes
     *
     * )                       → FIM do grupo de captura
     *
     * "                       → fecha a string JSON com aspas duplas
     *
     * Regex final (literal):  "chave"\s*:\s*"((?:[^"\\]|\\.)*)"
     * Em Java (String):       "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
     *
     * Por que (?:...) em vez de (...)?
     *   Grupos com () são numerados (grupo 1, grupo 2...) e podem ser referenciados no resultado.
     *   Grupos (?:) são usados apenas para estrutura lógica (operador | neste caso),
     *   sem criar um novo número de grupo. Isso mantém o grupo 1 sendo apenas o valor capturado.
     */
    Pattern padraoString = Pattern.compile(
            "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
    );
    Matcher m = padraoString.matcher(json);

    if (m.find()) {
        // m.group(1): Retorna o conteúdo do primeiro grupo de captura (os parênteses externos)
        // Isso é o valor da string JSON, mas ainda com os escapes do JSON (\", \\, \n)
        // Precisamos desfazê-los para obter o texto original

        return m.group(1)
                // A ORDEM IMPORTA: \" deve ser desescapado ANTES de \\
                // Se fizéssemos \\ primeiro, o \\ em \\\" viraria \ e a " seguinte não seria tocada
                .replace("\\\"", "\"")   // \" (aspas escapadas)   → "
                .replace("\\\\", "\\")   // \\ (barra dupla)        → \  (DEVE ser após o anterior)
                .replace("\\n",  "\n")   // \n (escape de nova linha) → caractere de nova linha real
                .replace("\\r",  "\r");  // \r (carriage return)    → caractere CR real
    }

    // ═══════════════════════════════════════════════════════════════════════════
    // CASO 2 — Campo do tipo número  (ex: "id": 1)
    // ═══════════════════════════════════════════════════════════════════════════

    /*
     * Regex: "chave"\s*:\s*(\d+)
     *
     *   \d    → dígito numérico (0-9)
     *   +     → um ou mais dígitos (garante que o número tem ao menos um dígito)
     *   ()    → grupo de captura — captura apenas os dígitos, sem aspas (números não têm aspas no JSON)
     *
     * Em Java: "\"" + chave + "\"\\s*:\\s*(\\d+)"
     */
    Pattern padraoNumero = Pattern.compile(
            "\"" + chave + "\"\\s*:\\s*(\\d+)"
    );
    m = padraoNumero.matcher(json);
    if (m.find()) {
        return m.group(1);  // retorna o número como String — o chamador faz Integer.parseInt() se necessário
    }

    return "";  // campo não encontrado no fragmento JSON
}
```

### 5.1. Decodificando a Regex de String Caractere por Caractere

```text
Regex (literal):  "chave"\s*:\s*"((?:[^"\\]|\\.)*)"

Exemplo de texto que ela vai varrer:
  "description": "Ler o livro \"Clean Code\""

Passo a passo do casamento:

1. "description"  →  casa o nome do campo entre aspas (literal)
         │
2. \s*:\s*        →  casa ": " (dois-pontos com espaço)
         │
3. "              →  casa a aspas de abertura do valor
         │
4. (              →  INICIA a captura do grupo 1
   │
   │  (?:         →  grupo de estrutura (não capturado)
   │   [^"\\]     →  casa "L", "e", "r", " ", "o", " ", "l", "i", "v", "r", "o", " "
   │   |          →
   │   \\.        →  casa \"  (barra + aspas — é um escape)
   │   )*         →  repete para cada caractere do valor
   │
5. )              →  FIM da captura → grupo(1) = 'Ler o livro \"Clean Code\"'
         │
6. "              →  casa a aspas de fechamento do valor

Resultado: m.group(1) = 'Ler o livro \"Clean Code\"'
Após replace("\\\"", "\""): 'Ler o livro "Clean Code"'  ← texto original restaurado
```

### 5.2. Código de Teste Isolado

#### Classe `JsonHandler.java` (módulo `extrairValor`)

```java
package tasktracker;

import java.util.regex.Matcher;  // Localiza correspondências de um Pattern em um texto
import java.util.regex.Pattern;  // Expressão regular compilada — reutilizável e eficiente

public class JsonHandler {

    // [MÉTODO DO ITEM 5]: Extrai o valor de um campo de um fragmento JSON usando regex
    // Suporta valores do tipo String (entre aspas) e número inteiro (sem aspas)
    // Acesso público aqui apenas para fins didáticos — em produção seria private
    public static String extrairValor(String json, String chave) {

        // ── Tentativa 1: o valor é uma String JSON (entre aspas duplas) ─────────
        // A regex captura o conteúdo entre as aspas, inclusive escapes como \" e \\
        Pattern padraoString = Pattern.compile(
                // "chave"  → nome do campo entre aspas (literal no JSON)
                "\"" + chave + "\""
                // \s*:\s*  → dois-pontos com espaços opcionais antes e depois
                + "\\s*:\\s*"
                // "        → abre as aspas do valor
                + "\""
                // (        → início do grupo de captura 1 (o valor que queremos)
                //   (?:    → grupo de estrutura (apenas agrupa, não cria novo número de grupo)
                //   [^"\\] → qualquer caractere que NÃO seja aspas nem barra invertida
                //   |      → OU
                //   \\.    → uma barra invertida seguida de qualquer outro caractere (sequência de escape)
                //   )*     → repete zero ou mais vezes (valor pode ser vazio "")
                // )        → fim do grupo de captura 1
                + "((?:[^\"\\\\]|\\\\.)*)"
                // "        → fecha as aspas do valor
                + "\""
        );

        Matcher m = padraoString.matcher(json);

        if (m.find()) {
            // group(1): Conteúdo do primeiro par de parênteses capturantes na regex
            // Ainda tem os escapes JSON (ex: \" para aspas) — precisamos desfazê-los
            return m.group(1)
                    // Ordem crítica: aspas escapadas ANTES de barras duplas
                    .replace("\\\"", "\"")  // \" no JSON → " no texto real
                    .replace("\\\\", "\\")  // \\ no JSON → \ no texto real
                    .replace("\\n",  "\n")  // \n no JSON → quebra de linha real
                    .replace("\\r",  "\r"); // \r no JSON → carriage return real
        }

        // ── Tentativa 2: o valor é um número inteiro (sem aspas no JSON) ────────
        // \d+ captura uma sequência de dígitos (0-9) com um ou mais caracteres
        Pattern padraoNumero = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*(\\d+)"
        );
        m = padraoNumero.matcher(json);
        if (m.find()) {
            return m.group(1);  // retorna como String — o chamador converte com Integer.parseInt()
        }

        return "";  // campo não encontrado ou JSON malformado
    }
}
```

#### Classe `Main.java` (visualização do `extrairValor`)

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {

        System.out.println("=== TESTANDO extrairValor() ===\n");

        // Bloco JSON de referência para todos os testes
        // Usa o mesmo formato que o arquivo tasks.json real produziria
        String blocoJson =
            "  {\n" +
            "    \"id\": 99,\n" +
            "    \"description\": \"Ler \\\"Design Patterns\\\" e tomar notas\",\n" +
            "    \"status\": \"in-progress\",\n" +
            "    \"createdAt\": \"2026-05-29 08:00:00\",\n" +
            "    \"updatedAt\": \"2026-05-29 11:45:00\"\n" +
            "  }";

        System.out.println("Fragmento JSON de entrada:");
        System.out.println("─────────────────────────────────────────");
        System.out.println(blocoJson);
        System.out.println("─────────────────────────────────────────\n");

        // ── Teste A: Extração de campo numérico (id) ──────────────────────────
        // A regex usada aqui é "id"\s*:\s*(\d+)  — sem aspas ao redor do valor
        String idStr = JsonHandler.extrairValor(blocoJson, "id");
        System.out.println("A) Campo 'id' extraído (String bruta): '" + idStr + "'");

        // Integer.parseInt(): Converte a String numérica para int primitivo
        int id = Integer.parseInt(idStr);
        System.out.println("   Após Integer.parseInt():            " + id + "  (tipo int)\n");

        // ── Teste B: Extração de campo String simples ─────────────────────────
        String status = JsonHandler.extrairValor(blocoJson, "status");
        System.out.println("B) Campo 'status': '" + status + "'\n");

        // ── Teste C: Extração de campo String com caracteres escapados ─────────
        // No JSON o valor está como:  "Ler \"Design Patterns\" e tomar notas"
        // Após o desescapamento:       Ler "Design Patterns" e tomar notas
        String descricao = JsonHandler.extrairValor(blocoJson, "description");
        System.out.println("C) Campo 'description' (com escape):");
        System.out.println("   Valor bruto no JSON: \"Ler \\\\\\\"Design Patterns\\\\\\\" e tomar notas\"");
        System.out.println("   Após desescapamento : '" + descricao + "'\n");

        // ── Teste D: Extração de campos de data ───────────────────────────────
        String createdAt  = JsonHandler.extrairValor(blocoJson, "createdAt");
        String updatedAt  = JsonHandler.extrairValor(blocoJson, "updatedAt");
        System.out.println("D) Campo 'createdAt': '" + createdAt + "'");
        System.out.println("   Campo 'updatedAt': '" + updatedAt + "'\n");

        // ── Teste E: Campo inexistente ────────────────────────────────────────
        // extrairValor() retorna "" (String vazia) quando o campo não é encontrado
        String campoInexistente = JsonHandler.extrairValor(blocoJson, "priority");
        System.out.println("E) Campo 'priority' (não existe):");
        System.out.println("   Retorno: '" + campoInexistente + "'");
        System.out.println("   Está vazio? " + campoInexistente.isEmpty());

        System.out.println("\n==========================================");
        System.out.println("Todos os tipos de campo extraídos com sucesso! ✅");
    }
}
```

### 5.3. Saída do Terminal (Output Esperado)

```text
=== TESTANDO extrairValor() ===

Fragmento JSON de entrada:
─────────────────────────────────────────
  {
    "id": 99,
    "description": "Ler \"Design Patterns\" e tomar notas",
    "status": "in-progress",
    "createdAt": "2026-05-29 08:00:00",
    "updatedAt": "2026-05-29 11:45:00"
  }
─────────────────────────────────────────

A) Campo 'id' extraído (String bruta): '99'
   Após Integer.parseInt():            99  (tipo int)

B) Campo 'status': 'in-progress'

C) Campo 'description' (com escape):
   Valor bruto no JSON: "Ler \"Design Patterns\" e tomar notas"
   Após desescapamento : 'Ler "Design Patterns" e tomar notas'

D) Campo 'createdAt': '2026-05-29 08:00:00'
   Campo 'updatedAt': '2026-05-29 11:45:00'

E) Campo 'priority' (não existe):
   Retorno: ''
   Está vazio? true

==========================================
Todos os tipos de campo extraídos com sucesso! ✅
Process finished with exit code 0
```

#### 5.3.1. Decodificando o Ciclo de Escape e Desescapamento

Uma das partes mais sutis da classe. Veja o ciclo completo que uma descrição com aspas percorre:

```text
TEXTO ORIGINAL (em memória, na Task):
  Ler "Design Patterns" e tomar notas

      ↓  Task.escaparJson() ao chamar toJson()

NO ARQUIVO tasks.json (em disco):
  "description": "Ler \"Design Patterns\" e tomar notas"
  As aspas internas viraram \" para não quebrar o JSON

      ↓  Files.readAllBytes() + new String(...)

CONTEÚDO LIDO (String Java em memória):
  "description": "Ler \"Design Patterns\" e tomar notas"
  (igual ao arquivo — ainda com os escapes)

      ↓  extrairValor() com Pattern.compile(...)

GRUPO CAPTURADO (m.group(1)):
  Ler \"Design Patterns\" e tomar notas
  (sem as aspas externas, mas ainda com \")

      ↓  .replace("\\\"", "\"")

TEXTO RESTAURADO (retornado ao chamador):
  Ler "Design Patterns" e tomar notas  ← idêntico ao original ✅
```

---

## 6. Teste de Integração Final — O Ciclo Completo

Agora que cada peça foi compreendida e testada isoladamente, vamos exercitar o ciclo completo: **criar tarefas → salvar → carregar → verificar integridade**.

Este teste prova que nenhuma informação se perde durante a serialização (Task → JSON) e a desserialização (JSON → Task).

### 6.1. O que será testado

```text
[Memória] Criar Tasks  →  saveTasks()  →  [Disco] tasks.json
                                                    │
                                                    ▼
[Memória] Comparar    ←  loadTasks()  ←  [Disco] tasks.json
```

### 6.2. Código do Teste de Integração

#### Classe `JsonHandler.java` (versão completa de produção)

```java
package tasktracker;

import java.io.File;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

// Versão completa e comentada da classe de produção
public class JsonHandler {

    // Caminho do arquivo JSON — relativo à pasta de execução do projeto
    private static final String CAMINHO_ARQUIVO = "tasks.json";

    // ─── Escrita ──────────────────────────────────────────────────────────────

    public static void saveTasks(List<Task> tarefas) {
        StringBuilder sb = new StringBuilder();
        sb.append("[\n");
        for (int i = 0; i < tarefas.size(); i++) {
            sb.append(tarefas.get(i).toJson());
            if (i < tarefas.size() - 1) {
                sb.append(",\n");
            }
        }
        sb.append("\n]");
        try {
            Files.write(
                    Paths.get(CAMINHO_ARQUIVO),
                    sb.toString().getBytes(StandardCharsets.UTF_8)
            );
        } catch (IOException e) {
            System.err.println("Erro ao salvar o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }
    }

    // ─── Leitura ──────────────────────────────────────────────────────────────

    public static List<Task> loadTasks() {
        List<Task> tarefas = new ArrayList<>();
        File arquivo = new File(CAMINHO_ARQUIVO);
        if (!arquivo.exists()) {
            return tarefas;
        }
        try {
            String conteudo = new String(
                    Files.readAllBytes(Paths.get(CAMINHO_ARQUIVO)),
                    StandardCharsets.UTF_8
            );
            Pattern padraoBlocoJson = Pattern.compile("\\{[^{}]+\\}", Pattern.DOTALL);
            Matcher matcher = padraoBlocoJson.matcher(conteudo);
            while (matcher.find()) {
                String blocoJson = matcher.group();
                Task tarefa = parsearTarefa(blocoJson);
                if (tarefa != null) {
                    tarefas.add(tarefa);
                }
            }
        } catch (IOException e) {
            System.err.println("Erro ao ler o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }
        return tarefas;
    }

    // ─── Parsing interno ──────────────────────────────────────────────────────

    private static Task parsearTarefa(String json) {
        try {
            int id              = Integer.parseInt(extrairValor(json, "id"));
            String descricao    = extrairValor(json, "description");
            String status       = extrairValor(json, "status");
            String criadoEm     = extrairValor(json, "createdAt");
            String atualizadoEm = extrairValor(json, "updatedAt");
            return new Task(id, descricao, status, criadoEm, atualizadoEm);
        } catch (NumberFormatException e) {
            System.err.println("Erro: ID inválido no JSON — " + e.getMessage());
            return null;
        } catch (Exception e) {
            System.err.println("Erro ao interpretar tarefa do JSON: " + e.getMessage());
            return null;
        }
    }

    private static String extrairValor(String json, String chave) {
        Pattern padraoString = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
        );
        Matcher m = padraoString.matcher(json);
        if (m.find()) {
            return m.group(1)
                    .replace("\\\"", "\"")
                    .replace("\\\\", "\\")
                    .replace("\\n",  "\n")
                    .replace("\\r",  "\r");
        }
        Pattern padraoNumero = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*(\\d+)"
        );
        m = padraoNumero.matcher(json);
        if (m.find()) return m.group(1);
        return "";
    }
}
```

#### Classe `Main.java` (teste de integração completo)

```java
package tasktracker;

import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {

        System.out.println("╔══════════════════════════════════════════════════════╗");
        System.out.println("║     TESTE DE INTEGRAÇÃO — JsonHandler Completo       ║");
        System.out.println("╚══════════════════════════════════════════════════════╝\n");

        // ── FASE 1: Criar tarefas em memória ─────────────────────────────────────

        System.out.println("FASE 1 — Criando tarefas em memória:\n");

        // Tarefa com descrição simples
        Task t1 = new Task(1, "Comprar leite");

        // Tarefa com descrição contendo aspas duplas — teste de escape
        Task t2 = new Task(2, "Ler o livro \"Clean Code\" (Cap. 1-3)");

        // Tarefa com descrição longa
        Task t3 = new Task(3, "Estudar expressões regulares em Java: Pattern e Matcher");

        // Tarefa com acentuação — teste de UTF-8
        Task t4 = new Task(4, "Revisar integração de configuração e lógica");

        // Alteramos status de algumas tarefas para testar variedade no JSON
        t2.setStatus("in-progress");
        t3.setStatus("done");

        // Monta a lista com todas as tarefas
        List<Task> tarefasOriginais = new ArrayList<>();
        tarefasOriginais.add(t1);
        tarefasOriginais.add(t2);
        tarefasOriginais.add(t3);
        tarefasOriginais.add(t4);

        for (Task t : tarefasOriginais) {
            System.out.println("  → " + t);
        }

        // ── FASE 2: Salvar no arquivo ─────────────────────────────────────────────

        System.out.println("\n─────────────────────────────────────────────────────────");
        System.out.println("FASE 2 — Persistindo no arquivo tasks.json:\n");

        JsonHandler.saveTasks(tarefasOriginais);

        System.out.println("  ✅ Arquivo tasks.json gravado com sucesso.\n");

        // ── FASE 3: Carregar do arquivo ───────────────────────────────────────────

        System.out.println("─────────────────────────────────────────────────────────");
        System.out.println("FASE 3 — Recarregando do arquivo (simulando nova execução):\n");

        // Simula o que acontece quando o programa reinicia:
        // tarefasOriginais saiu de escopo — só temos o arquivo em disco
        List<Task> tarefasCarregadas = JsonHandler.loadTasks();

        System.out.println("  ✅ " + tarefasCarregadas.size() + " tarefa(s) carregada(s).\n");

        for (Task t : tarefasCarregadas) {
            System.out.println("  ← " + t);
        }

        // ── FASE 4: Verificar integridade (comparar campo a campo) ───────────────

        System.out.println("\n─────────────────────────────────────────────────────────");
        System.out.println("FASE 4 — Verificando integridade dos dados:\n");

        boolean tudoOk = true;

        // Garante que a quantidade de tarefas é a mesma
        if (tarefasOriginais.size() != tarefasCarregadas.size()) {
            System.out.println("  ❌ Quantidade diverge! Original: " + tarefasOriginais.size()
                    + " | Carregadas: " + tarefasCarregadas.size());
            tudoOk = false;
        } else {
            // Compara campo a campo cada par de tarefas (original vs. carregada)
            for (int i = 0; i < tarefasOriginais.size(); i++) {
                Task orig = tarefasOriginais.get(i);
                Task load = tarefasCarregadas.get(i);

                boolean idOk     = orig.getId()          == load.getId();
                boolean descOk   = orig.getDescription().equals(load.getDescription());
                boolean statOk   = orig.getStatus()      .equals(load.getStatus());
                boolean criadoOk = orig.getCreatedAt()   .equals(load.getCreatedAt());
                boolean atualOk  = orig.getUpdatedAt()   .equals(load.getUpdatedAt());

                boolean tarefaOk = idOk && descOk && statOk && criadoOk && atualOk;
                tudoOk = tudoOk && tarefaOk;

                System.out.printf("  Tarefa %d: ID=%s | Desc=%s | Status=%s | CreatedAt=%s | UpdatedAt=%s → %s%n",
                        i + 1,
                        idOk     ? "✅" : "❌",
                        descOk   ? "✅" : "❌",
                        statOk   ? "✅" : "❌",
                        criadoOk ? "✅" : "❌",
                        atualOk  ? "✅" : "❌",
                        tarefaOk ? "OK" : "FALHOU"
                );
            }
        }

        System.out.println();
        if (tudoOk) {
            System.out.println("╔══════════════════════════════════════════════════════╗");
            System.out.println("║  ✅ TODOS OS DADOS PRESERVADOS COM PERFEIÇÃO!        ║");
            System.out.println("║  O ciclo serialização → disco → desserialização      ║");
            System.out.println("║  funcionou sem perda de nenhum campo.                ║");
            System.out.println("╚══════════════════════════════════════════════════════╝");
        } else {
            System.out.println("╔══════════════════════════════════════════════════════╗");
            System.out.println("║  ❌ ALGUM CAMPO DIVERGIU! Verifique os erros acima.  ║");
            System.out.println("╚══════════════════════════════════════════════════════╝");
        }
    }
}
```

### 6.3. Saída do Terminal (Output Esperado)

```text
╔══════════════════════════════════════════════════════╗
║     TESTE DE INTEGRAÇÃO — JsonHandler Completo       ║
╚══════════════════════════════════════════════════════╝

FASE 1 — Criando tarefas em memória:

  → [1] Comprar leite                              | Status: todo         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:00
  → [2] Ler o livro "Clean Code" (Cap. 1-3)        | Status: in-progress  | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01
  → [3] Estudar expressões regulares em Java: P... | Status: done         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01
  → [4] Revisar integração de configuração e ló... | Status: todo         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:00

─────────────────────────────────────────────────────────
FASE 2 — Persistindo no arquivo tasks.json:

  ✅ Arquivo tasks.json gravado com sucesso.

─────────────────────────────────────────────────────────
FASE 3 — Recarregando do arquivo (simulando nova execução):

  ✅ 4 tarefa(s) carregada(s).

  ← [1] Comprar leite                              | Status: todo         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:00
  ← [2] Ler o livro "Clean Code" (Cap. 1-3)        | Status: in-progress  | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01
  ← [3] Estudar expressões regulares em Java: P... | Status: done         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:01
  ← [4] Revisar integração de configuração e ló... | Status: todo         | Criado: 2026-05-29 10:00:00 | Atualizado: 2026-05-29 10:00:00

─────────────────────────────────────────────────────────
FASE 4 — Verificando integridade dos dados:

  Tarefa 1: ID=✅ | Desc=✅ | Status=✅ | CreatedAt=✅ | UpdatedAt=✅ → OK
  Tarefa 2: ID=✅ | Desc=✅ | Status=✅ | CreatedAt=✅ | UpdatedAt=✅ → OK
  Tarefa 3: ID=✅ | Desc=✅ | Status=✅ | CreatedAt=✅ | UpdatedAt=✅ → OK
  Tarefa 4: ID=✅ | Desc=✅ | Status=✅ | CreatedAt=✅ | UpdatedAt=✅ → OK

╔══════════════════════════════════════════════════════╗
║  ✅ TODOS OS DADOS PRESERVADOS COM PERFEIÇÃO!        ║
║  O ciclo serialização → disco → desserialização      ║
║  funcionou sem perda de nenhum campo.                ║
╚══════════════════════════════════════════════════════╝
Process finished with exit code 0
```

---

## Resumo dos Conceitos Aprendidos

| Conceito Java | Onde Foi Aplicado |
|---|---|
| `private static final` | Declaração da constante `CAMINHO_ARQUIVO` |
| `StringBuilder` | Montagem eficiente do JSON no `saveTasks` — evita criação de objetos intermediários |
| `Files.write()` | Gravação do arquivo em disco com criação automática se não existir |
| `Paths.get()` | Conversão de String em objeto `Path` entendível pela JVM |
| `StandardCharsets.UTF_8` | Garante que acentos e caracteres especiais sejam gravados e lidos corretamente |
| `File.exists()` | Verifica presença do arquivo antes de tentar ler (evita exceção na 1ª execução) |
| `Files.readAllBytes()` | Lê o arquivo inteiro de uma vez como array de bytes |
| `new String(byte[], Charset)` | Converte bytes lidos do disco em texto (String) |
| `Pattern.compile()` | Compila uma expressão regular em objeto reutilizável |
| `Pattern.DOTALL` | Flag que faz o ponto (`.`) casar com quebras de linha (`\n`) |
| `Matcher.find()` | Avança para a próxima ocorrência do padrão no texto |
| `Matcher.group()` | Retorna o trecho de texto que casou com o padrão |
| `Matcher.group(1)` | Retorna o conteúdo do primeiro grupo de captura `()` |
| `[^"\\]` | Classe negada — qualquer caractere exceto `"` e `\` |
| `(?:...)` | Grupo não-capturante — estrutura lógica sem criar novo número de grupo |
| `Integer.parseInt()` | Converte String numérica para o tipo primitivo `int` |
| `return null` | Sinaliza falha de parsing sem lançar exceção — isola erros por bloco |
| `catch (IOException e)` | Captura erros de leitura/escrita em disco (arquivo bloqueado, sem permissão...) |
| `System.err.println()` | Imprime no canal de erros padrão (vermelho no IntelliJ) |

---

## Mapa Mental da Classe

```text
                         JsonHandler
                              │
              ┌───────────────┼───────────────┐
              │               │               │
           (público)       (público)       (privado)
           saveTasks()     loadTasks()     parsearTarefa()
              │               │               │
              │               │           (privado)
              │               │           extrairValor()
              │               │               │
              ▼               ▼               │
         [tasks.json]    [tasks.json]         │
              │               │               │
       (escrita total)  (leitura total)       │
       sobrescreve      abre → lê →           │
       o arquivo        aplica regex ─────────┘
                        para cada bloco
```

---