```sh
mkdir backend001
cd backend001
dotnet new console

# Adicionar pacotes
dotnet add package Microsoft.EntityFrameworkCore -v 7.0.13
dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 7.0.13
dotnet add package Microsoft.EntityFrameworkCore.Design -v 7.0.13
dotnet add package Microsoft.EntityFrameworkCore.Tools.DotNet -v 2.0.3

# Listar pacotes
dotnet list package

# instalar pacotes pelo XML
dotnet restore

# Remover pacotes
dotnet remove package nome

# Limpar
dotnet clean

# Executar
dotnet run
```

# Versões do banco de dados

```sh
# Criar um migrations
dotnet ef migrations add Inicial

# Criar umj banco de dados
dotnet ef database update
```

.prettierrc.json

```json
{
	"trailingComma": "none",
	"tabWidth": 4,
	"semi": true,
	"singleQuote": true,
	"useTabs": true
}
```

omnisharp.json

```json
{
	"FormattingOptions": {
		"newLine": "\n",
		"useTabs": true,
		"tabSize": 4,
		"indentationSize": 4
	}
}
```

# VS Code - settings.json

```json
{
	"editor.defaultFormatter": "esbenp.prettier-vscode",
	"[csharp]": {
		"editor.defaultFormatter": "ms-dotnettools.csharp"
	}
}
```

Program.cs

```c#
using System;
using Microsoft.EntityFrameworkCore;
using System.Linq;
using System.Threading.Tasks;

namespace EFBackEnd001
{
	class Program
	{
		static async Task Main(string[] args)
		{
			using (var db = new livroContext())
			{
				db.Database.EnsureDeleted();
				db.Database.EnsureCreated();

				// ADD
				db.Livros.Add(new Livro { Titulo = "O Urubu que cagou no outro", Autor = "Clemas", AnoPublicacao = 2020 });
				db.Livros.Add(new Livro { Titulo = "To apaiando pra Caramba", Autor = "Clemas", AnoPublicacao = 2021 });
				db.Livros.Add(new Livro { Titulo = "O trem difícil", Autor = "Clemas", AnoPublicacao = 2021 });
				db.Livros.Add(new Livro { Titulo = "aaa", Autor = "Clemas", AnoPublicacao = 2021 });
				db.SaveChanges();

				// DEL - forma 1
				var lv = new Livro { LivroId = 4 };
				db.Livros.Attach(lv);
				db.Livros.Remove(lv);
				db.SaveChanges();

				// DEL - forma 2
				var rs = db.Livros.FirstOrDefault(l => l.LivroId == 5);
				if (rs != null){
					db.Livros.Remove(rs);
					db.SaveChanges();
				}

				// UPDATE
				rs = db.Livros.FirstOrDefault(l => l.LivroId == 6);
				if (rs != null){
					rs.Titulo = "TESTE TESTE";
					db.SaveChanges();
				}

				// Forma 1
				await db.Livros
				.ForEachAsync(l => Console.WriteLine($"{l.LivroId} | {l.Titulo} | {l.Autor}"));

				// forma 2
				var livros = await db.Livros.ToListAsync();
				foreach (var l in livros)
				{
					Console.WriteLine($"{l.LivroId} | {l.Titulo} | {l.Autor}");
				}

				// forma 3
				var livros = await db.Livros
				.OrderByDescending(l => l.LivroId)
				.Where(l => l.LivroId == 2 || l.LivroId == 1)
				.Skip(1)// similar limit of mysql
				.Take(1)// similar limit of mysql
				.Select(l => new { l.Titulo, l.Autor, l.LivroId})
				.ToListAsync();
				foreach (var l in livros)
				{
					Console.WriteLine($"{l.LivroId} | {l.Titulo} | {l.Autor}");
				}

				// forma 4 - sem await - count
				var owner = db.Livros.Where(l => l.LivroId > 0);
				db.Livros.Load();
				var count = db.Livros.Count();
				Console.WriteLine($"Count: {count}");

				// forma 5 - sem await
				db.Livros
				.OrderBy(l => l.LivroId)
				.OrderByDescending(l => l.LivroId)
				.Where(l => l.LivroId == 2)
				.Select(l => new { l.Titulo, l.Autor, l.LivroId})
				.ToList()
				.ForEach(l => Console.WriteLine($"{l.LivroId} | {l.Titulo} | {l.Autor}"));

			}
		}
	}
	public class livroContext : DbContext
	{
		public DbSet<Livro> Livros { get; set; }

		protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
		{
			// optionsBuilder.UseSqlServer(@"Server=localhost;Database=livros;Trusted_Connection=True;");
			optionsBuilder.UseSqlServer(@"Server=localhost\MSSQLSERVER01;Database=livros;Trusted_Connection=True;TrustServerCertificate=True;");
		}

		protected override void OnModelCreating(ModelBuilder modelBuilder)
		{
			modelBuilder.Entity<Livro>()
				.ToTable("Livro");

			modelBuilder.Entity<Livro>()
				.HasKey(p => p.LivroId);

			modelBuilder.Entity<Livro>()
				.Property(p => p.Titulo)
				.HasColumnType("varchar(50)")
				.IsRequired()
				.HasDefaultValue("");

			modelBuilder.Entity<Livro>()
				.Property(p => p.Autor)
				.HasColumnType("varchar(50)")
				.IsRequired()
				.HasDefaultValue("");

			modelBuilder.Entity<Livro>()
				.Property(p => p.AnoPublicacao)
				.HasColumnType("int")
				.IsRequired()
				.HasDefaultValue(0);
		}
	}

	public class Livro
	{
		public int? LivroId { get; set; }
		public string? Titulo { get; set; }
		public string? Autor { get; set; }
		public int? AnoPublicacao { get; set; }
	}
}
```
