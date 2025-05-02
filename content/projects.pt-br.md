+++
title = "Projetos"
slug = "projects"
+++

---

# [WordView](https://github.com/word-view) (2023 - Agora)
Uma plataforma de aprendizado de idiomas baseada em música, e atualmente meu principal foco.

A idéia é que o usuário possa escolher qualquer música internacional e enquanto ele escuta, o aplicativo identificará as palavras relevantes na letra e ao fim dela, gerará uma aula baseada nelas.

Além da parte de música, existem também planos para criar outra sessão baseada em livros.

## A estrutura
Existem dois principais projetos: APIWordView e WordView-android. E três bibliotecas complementares: Gengolex, Assis e WordFind.

###### [APIWordView](https://github.com/word-view/APIWordView)
API REST criada com Spring Boot que usa PostgreSQL como banco de dados. Ela oferece letras de músicas, aulas, imagens, autenticação de contas, e outras coisas necessárias ao app. Nela existem algumas pequenas adaptações feitas ao jeito que um projeto Spring Boot é estruturado que facilitam o desenvolvimento e deixam o código mais legível.


###### [WordView-android](https://github.com/word-view/WordView-android)
Atualmente o único front-end do projeto. O app é feito usando Kotlin com o toolkit Jetpack Compose. Possui uma home simples com algumas recomendações de música, uma barra de pesquisas onde você pode procurar por quaisquer músicas, um sistema de registrar ou fazer login em uma conta, e o player onde você escuta e lê as letras da música.

Existem alguns planos para no futuro transformá-lo em um projeto Kotlin Multiplatform, permitindo assim, que o app possa rodar no Android, IOS, na Web e localmente nos computadores, tudo com a mesma codebase.

Você pode ver algumas imagens do aplicativo na [galeria](gallery/#wordview-android).

###### [Gengolex](https://github.com/word-view/gengolex)
Pode ser definido como o "cérebro" da idéia. Para usá-lo o usuário deve criar um novo `Parser`, escolhendo um dos idiomas suportados pela biblioteca e passar um dicionário que poderá ser definido ou na pasta resources do projeto ou manualmente em formato de String. Os dicionários contém os registros das palavras, suas derivações, suas características, como ela deve ser apresentada ao usuário e etc. As palavras são identificas pelo atributo `parent` que serve de identificador global e que idealmente deve ser definido como a tradução da palavra em inglês, um exemplo de dicionário pode ser visto [aqui.](https://github.com/word-view/gengolex/blob/master/src/test/resources/dictionaries/kanji/kanji.json)

Enfim, o usuário pode processar uma String usando `Parser.findWords`, esta função retorna uma lista com todas as palavras presentes na String que também estão definidas no dicionário.

###### [Assis](https://github.com/word-view/assis)
Nomeado assim por conta do escrito Machado de Assis, é uma biblioteca feita em kotlin para ler ebooks. Atualmente não está sendo usada, mas será no futuro, na parte de livros do aplicativo.

A biblioteca consegue processar a maior parte dos arquivos epub, porém não é muito estável ainda. Para usá-la o usuario deve usar a função `parseEpub` e passar, como argumento, o epub como InputStream. O valor retornado por esta função é uma representação do livro que possui alguns métodos para ler o título, a capa, e o texto das páginas.

###### [WordFind](https://github.com/word-view/WordFind)
É uma biblioteca que pesquisa por letras de músicas em três diferentes plataformas escrita em Kotlin, pode ser extendida pelo usuário para acomodar mais plataformas.

Ela possui uma estrutura bem simples, cada classe que extende `Provider` tem um `rootUrl` e uma função `find`, o trabalho desta é montar o request, injetando os parâmetros da pesquisa no `rootUrl` e executá-lo. O request é então, passado para `parseResponse`, que decidi o que fazer baseado no status code, se for 200 OK, a letra da música é extraída e retornada para a classe principal.

Para adicionar uma plataforma, o usuário deverá criar sua própria subclasse de `Provider` e inseri-la no ArrayList `WordFind.platforms`, a biblioteca então percorre toda a lista até achar uma plataforma que tem a letra da música.

# [Quernel](https://github.com/64ArthurAraujo/Quernel) (2023)
Um simples kernel i686 feito para aprender C e Assembly. Inclui também uma libc simples.