# CodeRDIversity - API Lista de Tarefas

<img style = "width: 200px" src = "https://prosper.tech/wp-content/uploads/2023/02/stories_06-3-2-783x1024.png" alt = "CodeRDIversity"> <img style = "width: 150px" src = "https://media.licdn.com/dms/image/C4D0BAQESTAYGKhOdSQ/company-logo_200_200/0/1659620356007?e=2147483647&v=beta&t=9kHLR--f0dlKI6o6clQuGNllshlTOb96Mi51iU5idlg" alt = "Prosper Tech Talents"> <img style = "width: 200px" src = "https://www.rdisoftware.com/img/logo.png" alt = "RDI Softwares"> 

Exercício realizado durante a CodeRDIversity para o desenvolvimento de uma lista de tarefas.

## Sobre o Projeto
Realizado em Abril de 2023 durante o **CodeRDIversity** organizado pela **Prosper Tech Talents** em parceria com a **RDI Software** para o **treinamento** de **PCD's** na **linguagem C#**. O projeto foi proposto como exercício prático do curso, sendo orientado por **Rodrigo Grigoleto**, tendo como objetivo o desenvolvimento de uma API CRUD (Creat, Read, Update e Delete) de uma lista de tarefas.

## Tecnologias Utilizadas
- Linguagem C# / ASP Net
- .Net 6.0
- Sqlite
- Visual Stuio 2022 Community
- DBeaver
- Swagger / Postman

## Como executar o projeto
```bash
# clonar repositório
git clone https://github.com/PhilTisoni/CodeRDIversity--My_Book_Library.git
```
Após clonar o projeto, abra o executável na pasta bin.

# Índice

- <a href = "#Regra-de-Negócio">Regra de Negócio</a>
- <a href = "#Criação-do-Banco-de-Dados">Criação do Banco de Dados</a>
- <a href = "#Repositório">Repositório</a>
- <a href = "#Controller">Controller</a>
- <a href = "#Resultados">Resultados</a>- 
- <a href = "#Autores">Autores</a>
- <a href = "#Agradecimentos">Agradecimentos</a>

# Regra de Negócio

Crie uma API para o gerenciamento de uma lista de tarefas pessoal, como por exemplo:

- Estudar para prova de programação
- Estudar para a prova de matemática
- Verificar quando devo entregar o trabalho

As tarefas serão representadas por uma classe chamada **Tarefa** e deve possuir os seguintes atributos:

- Id (int)
- Descricao (string)
- DataCriacao (DateTime)
- Responsavel (string)
- Concluida (bool)

Deverá ser criada uma classe **Controller**, para que a API receba as requisições de manipulação das tarefas. A controller deverá possuir métodos para:

- Consultar todas as tarefas
- Consultar todas as tarefas concluídas
- Consultar todas as tarefas em aberto
- Incluir uma nova tarefa
- Atualizar a descrição de uma tarefa
- Excluir uma tarefa

Requisitos extras:

- Todas as tarefas deverão ser armazenadas em um **banco de dados SQlite**. Consequentemente, todas as manipulações das tarefas serão feitas com o banco;
- Coloque **DataAnnotation** na classe **Model**, para representar de maneira correta o nome da tabela e suas colunas no banco de dados;
- Separe os métodos de manipulação do banco de dados em classes do sufixo **Repository**;
- Teste todos os métodos no **Postman** - A utilização do Swagger é opcional.

# Criação do Banco de Dados

Para a criação do banco de dados, foram instalados os pacotes:

- **Microsoft.EntityFrameworkCore:** Instalaçõ do Entity Framework
- **Microsoft.EntityFrameworkCore.Design:** Utilização de Migrations
- **Microsoft.EntityFrameworkCore.Sqlite:** Utilização do Sqlite

Posteriormente, foi realizada a conexão com o banco através da classe **AppDbContext**:

```c#
  public class AppDbContext : DbContext
    {       
            public DbSet<Tarefa> Tarefas { get; set; }
        
            protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
            => optionsBuilder.UseSqlite(connectionString: "DataSource=app.db;Cache=Shared");
    }
```

Como requisitado na regra de negócio, foi criada uma classa Tarefa contendo os atributos citados utilizando DataAnnotation para a criação da tabela e das colunas no banco de dados:

```c#
    [Table("Tarefa")]
    public class Tarefa
    {
        [Column("id")]
        public int Id { get; set; }
        [Column("descricao")]
        public string Descricao { get; set; }
        [Column("dataCriacao")]
        public DateTime DataCriacao { get; set; }
        [Column("responsavel")]
        public string Responsavel { get; set; }
        [Column("isConcluido")]
        public bool IsConcluido { get; set; }
    }
```

# Repositório

Como forma de organização, foi criada uma pasta chamada **Repository** contendo uma classe com os métodos utilizados no CRUD através de injeção
de dependências. Abaixo, está exemplificado o método **ConsultarTodasTarefas()** que retorna uma lista com todas as tarefas cadastradas no banco:

```c#
        private readonly AppDbContext _context;

        public TarefaRepository(AppDbContext context)
        {
            _context = context;
        }

        public List<Tarefa> ConsultarTodasTarefas()
        {
            return _context.Tarefas.ToList();
        }
```

Para inserir o método de **ExcluirTarefa()**, foi adicionada uma validação através do método **ObterTarefaById()**:
```c#
       public void ExcluirTarefa(int id)
        {
            var tarefa = ObterTarefaById(id);
            if (tarefa != null)
            {
                _context.Tarefas.Remove(tarefa);
                _context.SaveChanges();
            }
        }

        public Tarefa ObterTarefaById(int id)
        {
            return _context.Tarefas.Find(id);
        }
```

# Controller

Para os métodos da Controller, foi criada uma **interface** na pasta Repository afim da utilização de injeção de dependências:
```c#
   public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            builder.Services.AddControllersWithViews();
            builder.Services.AddControllers();

            builder.Services.AddDbContext<AppDbContext>();
            builder.Services.AddTransient<ITarefaRepository, TarefaRepository>(); // Adicionando comando para injeção de dependências
```

Essa técnica facilitou a escrita e entendimento do código, simplificando os comandos para a Controller:

```c#
    [Route("[controller]")]
    public class ListaTarefasController : Controller
    {
        private readonly ITarefaRepository _tarefa;

        public ListaTarefasController(ITarefaRepository tarefa)
        {
            _tarefa = tarefa;
        }

        [HttpGet("[action]")]
        public List<Tarefa> ConsultarTodasTarefas()
        {
            return _tarefa.ConsultarTodasTarefas();
        }
```

Para os métodos Incluir e Atualizar tarefa, optou-se por utilizar a rota **FromQuery**, afim de simplificar o preenchimento dos dados no Swagger, porém, 
também seria possível a utilização de outras rotas, como o **FromRoute** ou **FromBody**, sendo necessários alguns ajustes na inserção desses comandos:
```c#  
        [HttpPost("[action]/{tarefa}")]
        public Tarefa IncluirTarefa([FromQuery] Tarefa tarefa)
        {
            return _tarefa.IncluirTarefa(tarefa);
        }
```


# Resultados

Utilizando o Swagger, podemos visualizar todos os comandos funcionando corretamente.


# Autores
- [Andreia Ribas](https://www.linkedin.com/in/andreiaribas/ "Andreia Linkedin")
- [Carolina Aizawa Moreira](https://www.linkedin.com/in/carolina-aizawa-moreira-pcd-9b0624179/ "Carolina Linkedin")
- Gabriel Vicente
- [Neto Trindade](https://www.linkedin.com/in/neto-trindade-09410287/ "Neto Linkedin")
- [Phelipe Augusto Tisoni](https://www.linkedin.com/in/phelipetisoni "Phelipe Linkedin")

# Agradecimentos
- [Rodrigo de Nadai Grigoleto](https://www.linkedin.com/in/rodrigo-de-nadai-grigoleto-58558133/ "Rodrigo Linkedin")
- [Prosper Tech Talents](https://www.linkedin.com/company/prosper-tech-talents/ "Prosper Linkedin")
- [RDI Software](https://www.linkedin.com/company/rdisoftware/ "RDI Linkedin")
