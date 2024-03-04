---
layout: post
title:  "Como fazer Web Crawling e Web Scraping com Node.js e o Puppeteer"
author: david
categories: [ Node.JS, JavaScript, Engenharia de Software]
image: assets/images/posts/nodejs-webcrawler.webp
comments: true
---

Todo mundo em algum momento já precisou "dar um Google", buscar alguma informação que precisava, e achar o site onde pudesse se aprofundar no conteúdo desejado. Porém algo que não paramos para pensar durante essas consultas na internet é como essa indexação de conteúdos de bilhões de sites da internet é feita, e como os sites de mecanismos de busca, ou seja Bing, Google, Yandex e entre outros fazem essa apresentação de diversos resultados para nós, e ainda mais, como ela funciona?
Neste artigo, vamos entender mais a fundo o que está "em baixo do capô" dos robôs que indexam conteúdos para os mecanismos de busca, e entender como o web crawling e web scraping são utilizados para as finalidades de indexação de conteúdos.

## O Puppeteer
O Puppeteer é uma produto para automação de browser, disponível para o Node.js e outras linguagens, ele fornece uma maneira fácil de controlar um navegador de forma automatizado. Com o Puppeteer, você pode criar scripts que navegam os sites da internet, e interagir com elementos da página, coletar informações e muito mais. Além disso, o Puppeteer é compatível com a maioria dos navegadores populares, como o Chrome, o Firefox e o Safari.

Com essas propriedades, o Puppeteer permite que possamos realizar web crawling e web scraping em sites da internet.

## Web Crawling vs. Web Scraping
Antes de começarmos, vamos esclarecer a diferença entre web crawling e web scraping.

O web crawling é o processo de coletar dados de vários sites da internet de maneira sistemática. É como se você estivesse rastreando a internet em busca de informações. O objetivo do web crawling é coletar o máximo de dados possível em um determinado assunto, seja para fins de pesquisa, análise de mercado ou outra finalidade.

Já o web scraping é o processo de extrair informações específicas de uma site da internet. O objetivo do web scraping é coletar dados relevantes de uma página da web para uma finalidade específica, como monitorar preços de produtos, coletar informações de contato, coletar dados de uma tabela, ou automatizar a tarefas que seriam realizadas de forma manual por um usuário.

Agora que sabemos a diferença entre web crawling e web scraping, vamos ver como usar o Puppeteer para realizar essas tarefas.

## Instalação do Puppeteer
Para começar, precisamos instalar o Puppeteer. Você pode instalar o Puppeteer usando o npm, o gerenciador de pacotes do Node.js.
```bash
npm install puppeteer
```
Depois de instalado, podemos começar a usar o Puppeteer em nossos scripts.

## Exemplo de Web Crawling
Vamos começar com um exemplo de web crawling. Neste exemplo, vamos usar o Puppeteer para coletar todas as URLs em uma página da web e, em seguida, seguir cada URL e coletar todas as URLs em cada página vinculada.

Para isso, vamos criar um arquivo `index.js` conforme a seguir.
```javascript
const puppeteer = require('puppeteer');

(async () => {

  //Abre uma sessão no browser e uma aba
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  const urls = new Set();
  const queue = new Set();

  const crawlingDepthLimit = 5;

  queue.add('https://www.medium.com/');

  while (queue.size > 0) {

    const url = queue.values().next().value;

    //Remove a próxima URL da fila e adiciona nos resultados
    queue.delete(url);
    urls.add(url);

    //Para de adicionar novas URLs na fila caso tenha chegado no limite configurado
    if (urls.size < crawlingDepthLimit){

      //Acessa a página informada da fila atual
      await page.goto(url);

      //Mapeia elementos HTML de links para adicionar a fila
      const newUrls = await page.$$eval('a', links =>
      links.map(link => link.href)
      );
  
      //Verificar duplicidades de URLs antes de adicionar na fila
      newUrls.forEach(newUrl => {
        if (!urls.has(newUrl) && !queue.has(newUrl)) {
          queue.add(newUrl);
        }

      });
    }
  }

  //Fecha o browser
  await browser.close();

  console.log([...urls]);

})();
```

Este script abre o site do [Medium](https://www.medium.com) em uma instância do navegador controlado pelo Puppeteer e coleta todas as URLs da página. Em seguida, adiciona cada URL à lista de URLs coletadas e adiciona todas as URLs recém-descobertas a uma fila para processamento posterior.

Em seguida, o script segue cada URL na fila e repete o processo, coletando todas as URLs em cada página vinculada e adicionando-as à fila. Como boa prática, foi adicionado um limite de crawling para evitar tempo de execução extensivo.
Executando esse script com Node.js, executando o comando node index.js temos o seguinte resultado:

![](assets/images/posts/puppeteer-result-01.png)

Já para termos um exemplo de script para web scraping, o script a seguir abre a página inicial do site [Example](https://www.example.com/) e extrai o texto do título da página (elemento `<h1>`) e o texto do primeiro parágrafo da página (elemento `<p>`). Em seguida, ele exibe essas informações no terminal.

```javascript
const puppeteer = require('puppeteer');

(async () => {

  //Abre uma sessão no browser e uma aba
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  //Acessa a página informada
  await page.goto('https://www.example.com/');

  //Coleta elementos HTML da página
  const title = await page.$eval('h1', el => el.innerText);
  const paragraph = await page.$eval('p', el => el.innerText);

  //Tira um print de tela do site no browser
  page.screenshot({path: `screenshot.png`})

  console.log(title);
  console.log(paragraph);

  //Fecha o browser
  await browser.close();
})();
```

Executando esse script com Node.js, executando o comando node index.js temos o seguinte resultado:

![](assets/images/posts/puppeteer-result-02.png)

Uma das funcionalidades é tirar capturas de tela da sessão do browser controlada pelo Puppeteer, como no exemplo a seguir com a página do [Example](https://www.example.com).

![](assets/images/posts/puppeteer-result-03.png)


Estes são apenas exemplos simples, mas com possibilidades ilimitadas, pois poderíamos extrair uma grande quantidade de informações úteis a partir de sites, tais como: preços de produtos, informações de contato, dados de tabelas, dentre outros. Assim como, realizar automações para tarefas repetitivas, ou testes de interface de usuário simulando um cliente final, onde poderíamos executar cliques em botões, e fazer uma experiência completa.

## Conclusão
O Puppeteer é uma ótima ferramenta para web crawling e web scraping em sites da internet. Ele nos permite automatizar ações no navegador, interagir com elementos da página e extrair informações específicas. Com o Puppeteer, podemos facilmente coletar dados de uma variedade de fontes, como sites de notícias, plataformas de mídia social, sites de comércio eletrônico e muito mais.

No entanto, é importante lembrar que o web scraping deve ser realizado de forma ética e legal. Alguns sites podem ter termos de serviço que proíbem o scraping de seus dados ou podem considerar o scraping de seus dados como uma violação de seus direitos autorais. Vale ressaltar também que muitos sites hoje em dia contém com mecanismos anti robôs, como o reCAPTCHA da Google por exemplo, que identifica comportamento de robôs como os que criamos acima, e realiza o bloqueio destes scripts.
Após a compreensão dos conceitos de web crawling e web scraping e seu uso, conseguimos entender como as informações dos sites da internet são extraídas e indexadas por mecanismos de busca, e entendendo que quando um determinado termo, texto ou frase é buscado (por exemplo, "O que é Node.js?"), o mecanismo busca na "última versão" indexada dos sites públicos da internet, que foram indexadas por robôs como os que criamos anteriormente.

Com a imagem a seguir, conseguimos visualizar diversas informações de vários sites da internet, e como os mecanismo de buscam usam o que foi indexado (ou seja, salva e manipula o HTML puro como fizemos acima) para trazer uma experiência customizada para o usuário.

![](assets/images/posts/exemplo-google-01.png)