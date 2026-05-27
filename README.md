# Task Tracker CLI

Aplicação de linha de comando (CLI) desenvolvida em **Java puro** para gerenciar suas tarefas.
Sem bibliotecas ou frameworks externos — apenas a Biblioteca Padrão do Java.

> Desenvolvido como parte do [desafio Task Tracker do roadmap.sh](https://roadmap.sh/projects/task-tracker).

---

## Índice

- [Funcionalidades](#funcionalidades)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Requisitos](#requisitos)
- [Instalação e Build](#instalação-e-build)
- [Como Usar](#como-usar)
- [Referência de Comandos](#referência-de-comandos)
- [Propriedades da Tarefa](#propriedades-da-tarefa)
- [Armazenamento de Dados](#armazenamento-de-dados)
- [Explicação do Código](#explicação-do-código)
- [Tratamento de Erros](#tratamento-de-erros)
- [Testes](#testes)

---

## Funcionalidades

- ✅ Adicionar, atualizar e remover tarefas
- ✅ Marcar tarefas como `in-progress` ou `done`
- ✅ Listar todas as tarefas ou filtrar por status
- ✅ Criação automática do arquivo JSON na primeira execução
- ✅ Persistência em `tasks.json` (sem banco de dados)
- ✅ Tratamento gracioso de todos os casos de erro
- ✅ IDs únicos e timestamps gerados automaticamente
- ✅ Sem dependências externas — Java Standard Library puro

---

## Estrutura do Projeto

```
task-tracker/
├── src/
│   └── tasktracker/
│       ├── Task.java           # Modelo de dados — representa uma tarefa
│       ├── JsonHandler.java    # Camada de persistência — lê/grava tasks.json
│       ├── TaskManager.java    # Lógica de negócio — todas as operações
│       └── Main.java           # Ponto de entrada — parser de argumentos CLI
├── out/
│   └── artifacts/
│       └── task_tracker_jar/
│           └── task-tracker.jar  # JAR executável compilado
└── tasks.json                  # Criado automaticamente na 1ª execução
```

---

## Requisitos

- **Java 11** ou superior
- Nenhuma biblioteca ou ferramenta de build externa (usa apenas `javac` / IntelliJ)

---

## Instalação e Build

### Opção 1 — IntelliJ IDEA

1. Abra o IntelliJ → **File → New Project** → Java
2. Crie o pacote `tasktracker` dentro de `src/`
3. Adicione os quatro arquivos fonte (veja a seção [Explicação do Código](#explicação-do-código))
4. **File → Project Structure → Artifacts → + → JAR → From modules with dependencies**
5. Defina a Main Class como `tasktracker.Main`
6. **Build → Build Artifacts → Build**

### Opção 2 — Linha de Comando (javac)

```bash
# Compilar
mkdir -p out
javac -d out src/tasktracker/*.java

# Executar diretamente
java -cp out tasktracker.Main add "Comprar leite"

# Ou criar um JAR
jar cfe task-tracker.jar tasktracker.Main -C out .
java -jar task-tracker.jar add "Comprar leite"
```

### Criando o alias `task-cli`

**Linux / macOS** — adicione ao `~/.bashrc` ou `~/.zshrc`:
```bash
alias task-cli='java -jar /caminho/completo/task-tracker.jar'
source ~/.bashrc
```

**Windows PowerShell** — adicione ao seu perfil PowerShell:
```powershell
function task-cli { java -jar "C:\caminho\completo\task-tracker.jar" @args }
```

---

## Como Usar

```bash
# Adicionando tarefas
task-cli add "Comprar leite"
# Saída: Tarefa adicionada com sucesso (ID: 1)

task-cli add "Ler o livro Clean Code"
# Saída: Tarefa adicionada com sucesso (ID: 2)

# Atualizando uma tarefa
task-cli update 1 "Comprar leite e cozinhar jantar"
# Saída: Tarefa 1 atualizada com sucesso.

# Removendo uma tarefa
task-cli delete 1
# Saída: Tarefa 1 removida com sucesso.

# Mudando o status
task-cli mark-in-progress 2
# Saída: Tarefa 2 atualizada: 'todo' → 'in-progress'.

task-cli mark-done 2
# Saída: Tarefa 2 atualizada: 'in-progress' → 'done'.

# Listando tarefas
task-cli list                # Todas as tarefas
task-cli list todo           # Apenas pendentes
task-cli list in-progress    # Apenas em andamento
task-cli list done           # Apenas concluídas
```

---

## Referência de Comandos

| Comando | Argumentos | Descrição |
|---------|------------|-----------|
| `add` | `"descrição"` | Cria uma nova tarefa com status `todo` |
| `update` | `<id> "descrição"` | Atualiza a descrição de uma tarefa existente |
| `delete` | `<id>` | Remove permanentemente uma tarefa |
| `mark-in-progress` | `<id>` | Define o status da tarefa como `in-progress` |
| `mark-done` | `<id>` | Define o status da tarefa como `done` |
| `list` | *(nenhum)* | Lista todas as tarefas |
| `list` | `todo` | Lista apenas tarefas com status `todo` |
| `list` | `in-progress` | Lista apenas tarefas com status `in-progress` |
| `list` | `done` | Lista apenas tarefas com status `done` |

---

## Propriedades da Tarefa

Cada tarefa armazenada no `tasks.json` possui os seguintes campos:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | Inteiro | Identificador único gerado automaticamente |
| `description` | String | Texto descrevendo o que precisa ser feito |
| `status` | String | Um dos valores: `todo`, `in-progress`, `done` |
| `createdAt` | String | Timestamp de criação da tarefa (`yyyy-MM-dd HH:mm:ss`) |
| `updatedAt` | String | Timestamp da última modificação (atualizado automaticamente) |

---

## Armazenamento de Dados

As tarefas são armazenadas em um arquivo `tasks.json` criado automaticamente no **diretório de execução** na primeira vez que o programa roda.

**Exemplo de `tasks.json`:**
```json
[
  {
    "id": 1,
    "description": "Comprar leite",
    "status": "done",
    "createdAt": "2024-01-15 09:00:00",
    "updatedAt": "2024-01-15 10:30:00"
  },
  {
    "id": 2,
    "description": "Ler o livro Clean Code",
    "status": "in-progress",
    "createdAt": "2024-01-15 09:05:00",
    "updatedAt": "2024-01-15 11:00:00"
  },
  {
    "id": 3,
    "description": "Fazer exercícios por 30 minutos",
    "status": "todo",
    "createdAt": "2024-01-15 09:10:00",
    "updatedAt": "2024-01-15 09:10:00"
  }
]
```

---

## Explicação do Código

### `Task.java` — Modelo de Dados

A classe `Task` é um objeto Java simples (POJO) que representa uma tarefa.

```java
package tasktracker;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Task {

    // Formato de data/hora compartilhado por todo o projeto
    public static final DateTimeFormatter FORMATTER =
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    private int id;
    private String description;
    private String status;       // "todo" | "in-progress" | "done"
    private String createdAt;    // Definido uma vez na criação, nunca alterado
    private String updatedAt;    // Atualizado automaticamente a cada modificação

    // ── Construtor para NOVAS tarefas ──────────────────────────────────────────
    // Define status como "todo" e ambos os timestamps com o horário atual
    public Task(int id, String description) {
        this.id = id;
        this.description = description;
        this.status = "todo";
        String agora = LocalDateTime.now().format(FORMATTER);
        this.createdAt = agora;
        this.updatedAt = agora;
    }

    // ── Construtor para CARREGAR tarefas do JSON ───────────────────────────────
    // Restaura todos os campos exatamente como foram salvos
    public Task(int id, String description, String status,
                String createdAt, String updatedAt) {
        this.id = id;
        this.description = description;
        this.status = status;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    // ── Getters ────────────────────────────────────────────────────────────────
    public int getId()             { return id; }
    public String getDescription() { return description; }
    public String getStatus()      { return status; }
    public String getCreatedAt()   { return createdAt; }
    public String getUpdatedAt()   { return updatedAt; }

    // ── Setters — atualizam updatedAt automaticamente ──────────────────────────
    public void setDescription(String description) {
        this.description = description;
        this.updatedAt = LocalDateTime.now().format(FORMATTER); // registra a mudança
    }

    public void setStatus(String status) {
        this.status = status;
        this.updatedAt = LocalDateTime.now().format(FORMATTER); // registra a mudança
    }

    // ── Serialização JSON (sem biblioteca externa) ─────────────────────────────
    // Produz um fragmento JSON pronto para ser gravado no tasks.json
    public String toJson() {
        return String.format(
                "  {\n" +
                "    \"id\": %d,\n" +
                "    \"description\": \"%s\",\n" +
                "    \"status\": \"%s\",\n" +
                "    \"createdAt\": \"%s\",\n" +
                "    \"updatedAt\": \"%s\"\n" +
                "  }",
                id, escaparJson(description), status, createdAt, updatedAt
        );
    }

    // Escapa caracteres especiais para produzir strings JSON válidas.
    // Sem isso, uma descrição como: ela disse "olá"
    // quebraria o JSON:             "description": "ela disse "olá""  ← inválido!
    private String escaparJson(String texto) {
        return texto
                .replace("\\", "\\\\")  // barra invertida — DEVE ser primeiro
                .replace("\"", "\\\"")  // aspas duplas
                .replace("\n", "\\n")   // nova linha
                .replace("\r", "\\r");  // retorno de carro
    }

    // ── Saída legível para o terminal ──────────────────────────────────────────
    @Override
    public String toString() {
        return String.format(
                "[%d] %-45s | Status: %-12s | Criado: %s | Atualizado: %s",
                id, description, status, createdAt, updatedAt
        );
    }
}
```

---

### `JsonHandler.java` — Camada de Persistência

Lê e grava o `tasks.json` usando **Java NIO** e **parsing JSON manual com expressões regulares**.
Nenhuma biblioteca externa (Jackson, Gson, etc.).

```java
package tasktracker;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.*;
import java.util.regex.*;

public class JsonHandler {

    private static final String CAMINHO_ARQUIVO = "tasks.json";

    // ── loadTasks() ────────────────────────────────────────────────────────────
    // Lê o arquivo JSON e retorna todas as tarefas como List<Task>.
    // Retorna lista vazia se o arquivo ainda não existir.
    public static List<Task> loadTasks() {
        List<Task> tarefas = new ArrayList<>();
        File arquivo = new File(CAMINHO_ARQUIVO);

        if (!arquivo.exists()) {
            return tarefas; // Primeira execução — arquivo ainda não existe, tudo certo
        }

        try {
            // Lê todo o conteúdo do arquivo como String UTF-8
            String conteudo = new String(
                    Files.readAllBytes(Paths.get(CAMINHO_ARQUIVO)),
                    StandardCharsets.UTF_8
            );

            // Regex para capturar cada objeto JSON { ... }
            // Pattern.DOTALL faz "." casar com quebras de linha também
            //
            // Detalhamento do padrão:  \{[^{}]+\}
            //   \{      → chave de abertura literal (escapada pois { tem sentido em regex)
            //   [^{}]+  → um ou mais caracteres que NÃO sejam chaves
            //             (evita capturar objetos aninhados acidentalmente)
            //   \}      → chave de fechamento literal
            Pattern padraoBlocoJson = Pattern.compile("\\{[^{}]+\\}", Pattern.DOTALL);
            Matcher matcher = padraoBlocoJson.matcher(conteudo);

            while (matcher.find()) {
                Task tarefa = parsearTarefa(matcher.group());
                if (tarefa != null) tarefas.add(tarefa);
            }

        } catch (IOException e) {
            System.err.println("Erro ao ler " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }

        return tarefas;
    }

    // ── saveTasks() ────────────────────────────────────────────────────────────
    // Grava a lista completa de tarefas no tasks.json, substituindo o conteúdo anterior.
    // Cria o arquivo se não existir.
    public static void saveTasks(List<Task> tarefas) {
        StringBuilder sb = new StringBuilder();
        sb.append("[\n");

        for (int i = 0; i < tarefas.size(); i++) {
            sb.append(tarefas.get(i).toJson());
            if (i < tarefas.size() - 1) sb.append(",\n"); // vírgula entre objetos, não após o último
        }

        sb.append("\n]");

        try {
            Files.write(
                    Paths.get(CAMINHO_ARQUIVO),
                    sb.toString().getBytes(StandardCharsets.UTF_8)
            );
        } catch (IOException e) {
            System.err.println("Erro ao gravar " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }
    }

    // ── parsearTarefa() ───────────────────────────────────────────────────────
    // Converte um fragmento de objeto JSON em um objeto Task.
    private static Task parsearTarefa(String json) {
        try {
            int id           = Integer.parseInt(extrairValor(json, "id"));
            String descricao = extrairValor(json, "description");
            String status    = extrairValor(json, "status");
            String criadoEm  = extrairValor(json, "createdAt");
            String atualizadoEm = extrairValor(json, "updatedAt");
            return new Task(id, descricao, status, criadoEm, atualizadoEm);
        } catch (Exception e) {
            System.err.println("Falha ao interpretar tarefa: " + e.getMessage());
            return null;
        }
    }

    // ── extrairValor() ────────────────────────────────────────────────────────
    // Extrai o valor de um campo nomeado de um fragmento JSON.
    // Suporta valores do tipo string ("chave": "valor") e inteiro ("chave": 42).
    private static String extrairValor(String json, String chave) {

        // Padrão para valor string:  "chave": "valor (pode conter \" e \\)"
        //
        // Regex:  "chave"\s*:\s*"((?:[^"\\]|\\.)*)"
        //   (?:[^"\\]|\\.)*  é um grupo não-capturante que casa:
        //     [^"\\]   → qualquer caractere exceto aspas e barra invertida
        //     \\.      → OU barra invertida seguida de qualquer caractere (sequência de escape)
        Pattern padraoString = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
        );
        Matcher m = padraoString.matcher(json);
        if (m.find()) {
            return m.group(1)
                    .replace("\\\"", "\"")  // desfaz \"  → "
                    .replace("\\\\", "\\")  // desfaz \\  → \  (DEVE ser após o anterior)
                    .replace("\\n", "\n")   // desfaz \n  → nova linha
                    .replace("\\r", "\r");  // desfaz \r  → retorno de carro
        }

        // Padrão para valor inteiro:  "chave": 42
        Pattern padraoNumero = Pattern.compile("\"" + chave + "\"\\s*:\\s*(\\d+)");
        m = padraoNumero.matcher(json);
        if (m.find()) return m.group(1);

        return ""; // campo não encontrado
    }
}
```

---

### `TaskManager.java` — Lógica de Negócio

Todas as operações sobre tarefas são implementadas aqui como métodos `public static`.
Cada método carrega o estado atual, executa a operação e salva de volta.

```java
package tasktracker;

import java.util.List;
import java.util.stream.Collectors;

public class TaskManager {

    // ── addTask() ─────────────────────────────────────────────────────────────
    // Adiciona nova tarefa. ID gerado como max(IDs existentes) + 1.
    // Se não houver tarefas ainda, o primeiro ID é 1.
    public static void addTask(String descricao) {
        if (descricao == null || descricao.trim().isEmpty()) {
            System.out.println("Erro: A descrição da tarefa não pode estar vazia.");
            return;
        }

        List<Task> tarefas = JsonHandler.loadTasks();

        // Stream: extrai todos os IDs → encontra o maior → soma 1
        // orElse(0) trata o caso de lista vazia: 0 + 1 = primeiro ID é 1
        int novoId = tarefas.stream()
                .mapToInt(Task::getId)   // Task::getId é uma referência de método
                .max()
                .orElse(0) + 1;

        tarefas.add(new Task(novoId, descricao.trim()));
        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa adicionada com sucesso (ID: " + novoId + ")");
    }

    // ── updateTask() ──────────────────────────────────────────────────────────
    // Atualiza a descrição de uma tarefa existente pelo ID.
    // setDescription() atualiza updatedAt automaticamente.
    public static void updateTask(int id, String novaDescricao) {
        if (novaDescricao == null || novaDescricao.trim().isEmpty()) {
            System.out.println("Erro: A nova descrição não pode estar vazia.");
            return;
        }

        List<Task> tarefas = JsonHandler.loadTasks();
        Task tarefa = buscarPorId(tarefas, id);

        if (tarefa == null) {
            System.out.println("Erro: Nenhuma tarefa encontrada com ID " + id + ".");
            return;
        }

        tarefa.setDescription(novaDescricao.trim());
        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa " + id + " atualizada com sucesso.");
    }

    // ── deleteTask() ──────────────────────────────────────────────────────────
    // Remove permanentemente uma tarefa pelo ID.
    // removeIf() retorna true se algum elemento foi removido, false caso contrário.
    public static void deleteTask(int id) {
        List<Task> tarefas = JsonHandler.loadTasks();
        boolean removeu = tarefas.removeIf(t -> t.getId() == id);

        if (!removeu) {
            System.out.println("Erro: Nenhuma tarefa encontrada com ID " + id + ".");
            return;
        }

        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa " + id + " removida com sucesso.");
    }

    // ── markInProgress() / markDone() ─────────────────────────────────────────
    // Ambos delegam para o método auxiliar privado alterarStatus().
    public static void markInProgress(int id) { alterarStatus(id, "in-progress"); }
    public static void markDone(int id)        { alterarStatus(id, "done"); }

    // ── listTasks() ───────────────────────────────────────────────────────────
    // Lista as tarefas no terminal em formato de tabela.
    // Se filtroStatus for null ou vazio → lista todas as tarefas.
    // Caso contrário → filtra pelo status fornecido ("todo", "in-progress", "done").
    public static void listTasks(String filtroStatus) {
        List<Task> tarefas = JsonHandler.loadTasks();
        List<Task> resultado;

        if (filtroStatus == null || filtroStatus.trim().isEmpty()) {
            resultado = tarefas; // sem filtro — exibe todas
        } else {
            // Valida se o filtro é um status reconhecido
            if (!filtroStatus.equals("todo")
                    && !filtroStatus.equals("in-progress")
                    && !filtroStatus.equals("done")) {
                System.out.println("Erro: Status inválido '" + filtroStatus + "'.");
                System.out.println("      Use: todo | in-progress | done");
                return;
            }

            // Stream filter: mantém apenas tarefas com o status solicitado
            resultado = tarefas.stream()
                    .filter(t -> t.getStatus().equals(filtroStatus))
                    .collect(Collectors.toList());
        }

        if (resultado.isEmpty()) {
            System.out.println("Nenhuma tarefa encontrada" +
                    (filtroStatus != null && !filtroStatus.isEmpty()
                            ? " com status '" + filtroStatus + "'" : "") + ".");
            return;
        }

        // Cabeçalho da tabela
        String linha = "─".repeat(110);
        System.out.println(linha);
        System.out.printf("%-5s %-45s %-14s %-21s %-21s%n",
                "ID", "Descrição", "Status", "Criado em", "Atualizado em");
        System.out.println(linha);

        // Uma linha por tarefa
        resultado.forEach(t -> System.out.printf(
                "%-5d %-45s %-14s %-21s %-21s%n",
                t.getId(), t.getDescription(), t.getStatus(),
                t.getCreatedAt(), t.getUpdatedAt()));

        System.out.println(linha);
        System.out.println("Total: " + resultado.size() + " tarefa(s)" +
                (filtroStatus != null && !filtroStatus.isEmpty()
                        ? " com status '" + filtroStatus + "'" : "") + ".");
    }

    // ── Métodos auxiliares privados ───────────────────────────────────────────

    // Altera o status de uma tarefa pelo ID.
    // Exibe uma mensagem de transição: 'antigo' → 'novo'.
    private static void alterarStatus(int id, String novoStatus) {
        List<Task> tarefas = JsonHandler.loadTasks();
        Task tarefa = buscarPorId(tarefas, id);

        if (tarefa == null) {
            System.out.println("Erro: Nenhuma tarefa encontrada com ID " + id + ".");
            return;
        }

        String anterior = tarefa.getStatus();
        tarefa.setStatus(novoStatus);
        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa " + id + " atualizada: '" + anterior + "' → '" + novoStatus + "'.");
    }

    // Busca uma tarefa pelo ID usando a Stream API.
    // Retorna null se não encontrar (via Optional.orElse(null)).
    private static Task buscarPorId(List<Task> tarefas, int id) {
        return tarefas.stream()
                .filter(t -> t.getId() == id)
                .findFirst()
                .orElse(null);
    }
}
```

---

### `Main.java` — Ponto de Entrada CLI

Faz o parsing dos argumentos da linha de comando e despacha para `TaskManager`.

```java
package tasktracker;

public class Main {

    public static void main(String[] args) {

        if (args.length == 0) {
            exibirAjuda();
            return;
        }

        String comando = args[0].toLowerCase();

        try {
            switch (comando) {

                case "add":
                    // Requer: args[1] = "descrição"
                    if (args.length < 2) { System.out.println("Uso: task-cli add \"descrição\""); return; }
                    TaskManager.addTask(args[1]);
                    break;

                case "update":
                    // Requer: args[1] = id (inteiro), args[2] = "nova descrição"
                    if (args.length < 3) { System.out.println("Uso: task-cli update <id> \"descrição\""); return; }
                    TaskManager.updateTask(parseInt(args[1]), args[2]);
                    break;

                case "delete":
                    // Requer: args[1] = id (inteiro)
                    if (args.length < 2) { System.out.println("Uso: task-cli delete <id>"); return; }
                    TaskManager.deleteTask(parseInt(args[1]));
                    break;

                case "mark-in-progress":
                    // Requer: args[1] = id (inteiro)
                    if (args.length < 2) { System.out.println("Uso: task-cli mark-in-progress <id>"); return; }
                    TaskManager.markInProgress(parseInt(args[1]));
                    break;

                case "mark-done":
                    // Requer: args[1] = id (inteiro)
                    if (args.length < 2) { System.out.println("Uso: task-cli mark-done <id>"); return; }
                    TaskManager.markDone(parseInt(args[1]));
                    break;

                case "list":
                    // Opcional: args[1] = "todo" | "in-progress" | "done"
                    // Se args[1] ausente → lista todas as tarefas
                    TaskManager.listTasks(args.length >= 2 ? args[1] : null);
                    break;

                default:
                    System.out.println("Comando desconhecido: '" + comando + "'");
                    exibirAjuda();
                    break;
            }

        } catch (IllegalArgumentException e) {
            System.out.println("Erro: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Erro inesperado: " + e.getMessage());
        }
    }

    // Converte uma String para int positivo com mensagem de erro amigável.
    private static int parseInt(String texto) {
        try {
            int valor = Integer.parseInt(texto);
            if (valor <= 0) throw new IllegalArgumentException(
                    "O ID deve ser um número inteiro positivo. Recebido: " + texto);
            return valor;
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException(
                    "ID inválido: '" + texto + "'. O ID deve ser um número inteiro.");
        }
    }

    // Exibe o texto de ajuda completo com todos os comandos disponíveis.
    private static void exibirAjuda() {
        System.out.println();
        System.out.println("╔══════════════════════════════════════════════════════╗");
        System.out.println("║           Task Tracker CLI — Ajuda                   ║");
        System.out.println("╚══════════════════════════════════════════════════════╝");
        System.out.println();
        System.out.println("USO:  task-cli <comando> [argumentos]");
        System.out.println();
        System.out.println("COMANDOS:");
        System.out.println("  add \"descrição\"              Adiciona uma nova tarefa");
        System.out.println("  update <id> \"descrição\"      Atualiza a descrição");
        System.out.println("  delete <id>                  Remove uma tarefa");
        System.out.println("  mark-in-progress <id>        Marca como em andamento");
        System.out.println("  mark-done <id>               Marca como concluída");
        System.out.println("  list                         Lista todas as tarefas");
        System.out.println("  list todo                    Lista tarefas pendentes");
        System.out.println("  list in-progress             Lista tarefas em andamento");
        System.out.println("  list done                    Lista tarefas concluídas");
        System.out.println();
        System.out.println("EXEMPLOS:");
        System.out.println("  task-cli add \"Comprar leite\"");
        System.out.println("  task-cli update 1 \"Comprar leite e ovos\"");
        System.out.println("  task-cli mark-in-progress 2");
        System.out.println("  task-cli list done");
        System.out.println("  task-cli delete 3");
        System.out.println();
    }
}
```

---

## Tratamento de Erros

A aplicação trata todos os cenários de erro de forma elegante (sem stack trace para o usuário):

| Cenário | Resposta |
|---------|----------|
| Nenhum comando fornecido | Exibe o texto de ajuda completo |
| Comando desconhecido | Exibe "Comando desconhecido" + ajuda |
| Argumento obrigatório ausente | Exibe dica de uso para aquele comando |
| ID não numérico (ex: `task-cli delete abc`) | `"Erro: ID inválido: 'abc'..."` |
| ID não positivo (ex: `task-cli delete 0`) | `"Erro: O ID deve ser um número positivo..."` |
| ID não encontrado (ex: `task-cli delete 99`) | `"Erro: Nenhuma tarefa encontrada com ID 99."` |
| Descrição vazia (ex: `task-cli add ""`) | `"Erro: A descrição não pode estar vazia."` |
| Filtro de status inválido (ex: `task-cli list xyz`) | `"Erro: Status inválido 'xyz'..."` |
| Erro de leitura/escrita no arquivo | Imprime no `System.err`, continua sem travar |

---

## Testes

### Sequência completa de testes

```bash
# 1. Começar do zero
rm -f tasks.json

# 2. Adicionar tarefas
task-cli add "Comprar leite"
# → Tarefa adicionada com sucesso (ID: 1)

task-cli add "Estudar Java Streams"
# → Tarefa adicionada com sucesso (ID: 2)

task-cli add "Fazer 30 minutos de exercício"
# → Tarefa adicionada com sucesso (ID: 3)

task-cli add "Ler Clean Code"
# → Tarefa adicionada com sucesso (ID: 4)

# 3. Listar todas
task-cli list
# → Exibe 4 tarefas, todas com status "todo"

# 4. Alterar status
task-cli mark-in-progress 2
# → Tarefa 2 atualizada: 'todo' → 'in-progress'.

task-cli mark-done 1
# → Tarefa 1 atualizada: 'todo' → 'done'.

task-cli mark-done 3
# → Tarefa 3 atualizada: 'todo' → 'done'.

# 5. Filtrar por status
task-cli list done
# → Exibe tarefas 1 e 3

task-cli list in-progress
# → Exibe tarefa 2

task-cli list todo
# → Exibe tarefa 4

# 6. Atualizar descrição
task-cli update 4 "Ler Clean Code e refatorar o projeto"
# → Tarefa 4 atualizada com sucesso.

# 7. Remover tarefa
task-cli delete 1
# → Tarefa 1 removida com sucesso.

# 8. Casos de erro
task-cli delete 99
# → Erro: Nenhuma tarefa encontrada com ID 99.

task-cli add ""
# → Erro: A descrição da tarefa não pode estar vazia.

task-cli list xyz
# → Erro: Status inválido 'xyz'. Use: todo | in-progress | done

task-cli update abc "teste"
# → Erro: ID inválido: 'abc'. O ID deve ser um número inteiro.

# 9. Estado final
task-cli list
# → Exibe tarefas 2, 3 e 4 com seus status atuais
```

### Estado esperado do `tasks.json` ao final

```json
[
  {
    "id": 2,
    "description": "Estudar Java Streams",
    "status": "in-progress",
    "createdAt": "2024-01-15 09:01:00",
    "updatedAt": "2024-01-15 09:05:00"
  },
  {
    "id": 3,
    "description": "Fazer 30 minutos de exercício",
    "status": "done",
    "createdAt": "2024-01-15 09:02:00",
    "updatedAt": "2024-01-15 09:06:00"
  },
  {
    "id": 4,
    "description": "Ler Clean Code e refatorar o projeto",
    "status": "todo",
    "createdAt": "2024-01-15 09:03:00",
    "updatedAt": "2024-01-15 09:07:00"
  }
]
```

---

## Conceitos Java Utilizados

| Conceito | Onde Foi Aplicado |
|----------|-------------------|
| Classes e encapsulamento | `Task.java` — atributos privados, getters/setters públicos |
| Construtores sobrecarregados | `Task.java` — dois construtores com assinaturas diferentes |
| `LocalDateTime` e `DateTimeFormatter` | `Task.java` — geração e formatação de timestamps |
| `java.nio.file.Files` | `JsonHandler.java` — API moderna de I/O de arquivos |
| Expressões regulares (`Pattern`, `Matcher`) | `JsonHandler.java` — parsing JSON manual |
| `Pattern.DOTALL` | `JsonHandler.java` — faz `.` casar com quebras de linha |
| `StringBuilder` | `JsonHandler.java` — construção eficiente de strings longas |
| Stream API (`filter`, `mapToInt`, `max`, `findFirst`, `collect`) | `TaskManager.java` |
| Expressões lambda | `TaskManager.java` — `t -> t.getId() == id` |
| Referências de método | `TaskManager.java` — `Task::getId` |
| `Optional` | `TaskManager.java` — resultado de `findFirst()` / `max()` |
| `List.removeIf()` | `TaskManager.java` — remoção condicional de elementos |
| Instrução `switch` | `Main.java` — despacho de comandos |
| Tratamento de exceções | `Main.java` — try/catch para `NumberFormatException` |
| `String.format()` | Em todo o projeto — saída formatada com `%d`, `%s`, `%-45s` |

---

## Licença

Este projeto foi desenvolvido como exercício de aprendizado para o [desafio Task Tracker do roadmap.sh](https://roadmap.sh/projects/task-tracker).
Sinta-se à vontade para usar, modificar e compartilhar livremente.