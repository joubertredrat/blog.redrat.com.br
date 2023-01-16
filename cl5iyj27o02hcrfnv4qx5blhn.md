# Gerenciando as variáveis de ambiente do seu projeto Golang

Olá pessoas, neste artigo iremos falar sobre uma maneira fácil que eu uso para gerenciar as variáveis de ambiente nos meus projetos Golang, porém, vamos primeiro para os alertas.

#### Este artigo não é uma cagação de regra
Sim, as vezes é bom lembrar que o artigo é apenas para fins educativos, para passar o conhecimento adiante e levantar o debate, não é uma cagação de regra.

#### O problema
Por várias vezes eu me deparei com projetos em golang em que tem o seguinte código com o [os.Getenv](https://pkg.go.dev/os#Getenv), como no exemplo abaixo.
```go
func dsn() string {
	return fmt.Sprintf(
		"postgres://%s:%s@%s:%s/%s?sslmode=%s",
		os.Getenv("DATABASE_USERNAME"),
		os.Getenv("DATABASE_PASSWORD"),
		os.Getenv("DATABASE_HOST"),
		os.Getenv("DATABASE_PORT"),
		os.Getenv("DATABASE_DBNAME"),
		os.Getenv("DATABASE_SSL_MODE"),
	)
}
```
Ok, mas qual o problema então? O problema é que esse código está acoplado, o que torna difícil testar, por exemplo. Outro problema é que não seria interessante haver chamadas ao `os.Getenv` espalhado pelo código, tornaria ele difícil de manter, como já aconteceu comigo antes.

Uma forma fácil de resolver este problema é justamente a Injeção de dependência, o que torna muito mais limpo e organizado o código, já que suas funções, structs e outros terão apenas os recursos que precisam para funcionar, então, pela lógica, o código acima ficaria como o exemplo abaixo.
```go
func dsn(host, port, user, pass, name, sslmode string) string {
	return fmt.Sprintf(
		"postgres://%s:%s@%s:%s/%s?sslmode=%s", 
		user, 
		pass, 
		host, 
		port, 
		name, 
		sslmode,
	)
}
```
Porém, isto agora cria um novo problema, como obter os dados que estão disponíveis via `os.Getenv`? 

#### Você no controle das variáveis de ambiente do projeto

Hoje, você pode definir as variáveis de ambiente de duas formas, em um arquivo `.env` ou definindo elas direto no seu shell, então para termos controle sobre isto, iremos usar 2 bibliotecas em Golang disponíveis no GitHub, que são as que estão abaixo.

* [https://github.com/joho/godotenv](https://github.com/joho/godotenv).
* [https://github.com/caarlos0/env](https://github.com/caarlos0/env).

A primeira biblioteca permite carregar as variáveis de ambiente de um arquivo `.env`, enquanto a segunda, permite fazer o parser das variáveis de ambiente para uma struct.

Com isto, poderemos ter nossa struct para fazer o gerenciamento dos dados das nossas variáveis de ambiente, como abaixo.
```go
type DbSSLMode string

type Config struct {
	ApiEnv           string    `env:"API_ENV,required"`
	ApiHost          string    `env:"API_HOST,required"`
	ApiPort          string    `env:"API_PORT,required"`
	DatabaseHost     string    `env:"DATABASE_HOST,required"`
	DatabasePort     string    `env:"DATABASE_PORT,required"`
	DatabaseUsername string    `env:"DATABASE_USERNAME,required"`
	DatabasePassword string    `env:"DATABASE_PASSWORD,required"`
	DatabaseDBName   string    `env:"DATABASE_DBNAME,required"`
	DatabaseSSLMode  DbSSLMode `env:"DATABASE_SSL_MODE,required"`
}
```
Note que nesta struct temos tags para referenciar a origem dos dados e também quais são obrigatórias e sim, isto é importante, porque acreditem ou não, já vi APIs quebrarem em produção porque justamente um `os.Getenv` perdido no projeto não teve sua variável de ambiente correspondente definida e isto estava derrubando a API.

Caso você tenha um arquivo `.env`, você simplesmente pode carregar ele, como abaixo.
```go
const (
	ENV_FILE               = ".env"
)

func loadEnvFile() error {
	if _, err := os.Stat(ENV_FILE); os.IsNotExist(err) {
		return nil
	}

	return godotenv.Load(ENV_FILE)
}
```
Com isto você pode ter a flexibilidade de usar um arquivo `.env` caso esteja desenvolvendo seu projeto.

Por fim, temos constantes e funções para você poder fazer validações que você ache interessante para também não ter surpresinhas quando seu projeto for para produção.
```go
const (
	ENV_DEV                = "dev"
	ENV_STAGING            = "staging"
	ENV_PROD               = "prod"
	DB_SSLMODE_DISABLE     = "disable"
	DB_SSLMODE_ALLOW       = "allow"
	DB_SSLMODE_PREFER      = "prefer"
	DB_SSLMODE_REQUIRE     = "require"
	DB_SSLMODE_VERIFY_CA   = "verify-ca"
	DB_SSLMODE_VERIFY_FULL = "verify-full"
)

func envModes() []string {
	return []string{ENV_DEV, ENV_STAGING, ENV_PROD}
}

func dbSSLModes() []string {
	return []string{
		DB_SSLMODE_DISABLE,
		DB_SSLMODE_ALLOW,
		DB_SSLMODE_PREFER,
		DB_SSLMODE_REQUIRE,
		DB_SSLMODE_VERIFY_CA,
		DB_SSLMODE_VERIFY_FULL,
	}
}
```

Agora vamos para a parte interessante, como obter essas os dados das variáveis de ambiente.
```go
func GetConfig() (Config, error) {
	if err := loadEnvFile(); err != nil {
		return Config{}, err
	}

	customParsers := map[reflect.Type]env.ParserFunc{
		reflect.TypeOf(DbSSLMode("")): func(v string) (interface{}, error) {
			for _, sslmode := range dbSSLModes() {
				if sslmode == string(v) {
					return DbSSLMode(v), nil
				}
			}

			return nil, errors.New(fmt.Sprintf(
				`Invalid environment variable "DATABASE_SSL_MODE" %s, available options are: %s`,
				v,
				strings.Join(dbSSLModes(), ", "),
			))
		},
	}

	config := Config{}
	if err := env.ParseWithFuncs(&config, customParsers); err != nil {
		return Config{}, err
	}

	if !slices.Contains(envModes(), config.ApiEnv) {
		return Config{}, errors.New(fmt.Sprintf(
			`Invalid environment variable "API_ENV" %s, available options are: %s`,
			config.ApiEnv,
			strings.Join(envModes(), ", "),
		))
	}

	return config, nil
}
```
Na função acima, iremos carregar o arquivo `.env`, caso você tenha ele na raiz do seu projeto, cria um parser customizado caso queira, sendo que no exemplo, é o SSL Mode do banco de dados, que tem opções estritas.
Depois disto, basta criar a struct, fazer o parse e pronto! Depois disto você ainda pode fazer validações também, tudo para manter o controle sobre os dados das suas variáveis de ambiente.

#### Executando o projeto

Uma vez criado sua struct com as validações, basta chamar a função e tratar o erro, caso algo esteja errado, como nos exemplos abaixo.
```go
	config, err := pkg.GetConfig()
	if err != nil {
		panic(err)
	}
```
```
dev@Devs-MacBook-Pro ~/source/joubertredrat/go-env-management> go run main.go
panic: env: required environment variable "API_PORT" is not set

dev@Devs-MacBook-Pro ~/source/joubertredrat/go-env-management> go run main.go
panic: Invalid environment variable "API_ENV" testing, available options are: dev, staging, prod

dev@Devs-MacBook-Pro ~/source/joubertredrat/go-env-management> go run main.go
panic: env: parse error on field "DatabaseSSLMode" of type "pkg.DbSSLMode": Invalid environment variable "DATABASE_SSL_MODE" deny, available options are: disable, allow, prefer, require, verify-ca, verify-full
```

Por fim, corrigidos todas as variáveis de ambiente, você terá uma struct com os dados para usar nas funções, métodos e outros que precisar.
```go
func main() {
	config, err := pkg.GetConfig()
	if err != nil {
		panic(err)
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		io.WriteString(w, dsn(
			config.DatabaseHost,
			config.DatabasePort,
			config.DatabaseUsername,
			config.DatabasePassword,
			config.DatabaseDBName,
			string(config.DatabaseSSLMode),
		))
	})

	listen := fmt.Sprintf("%s:%s", config.ApiHost, config.ApiPort)
	fmt.Printf("Running app %s at %s\n", config.ApiEnv, listen)

	if err := http.ListenAndServe(listen, nil); err != nil {
		panic(err)
	}
}
```

#### Projeto em ação

Um exemplo completo desta abordagem funcionando está no Github abaixo.

* [https://github.com/joubertredrat/go-env-management](https://github.com/joubertredrat/go-env-management).

Então é isto, espero ter ajudado, até a próxima!