@@@@@
esse-blog-e-feito-com-javascript-puro | [Rascunho] Esse blog é feito com JavaScript puro | ptbr | 2024-09-21T18:14:57.044Z | Criando um blog modularizado dinâmico e moderno com JavaScript puro, sem TS ou frameworks.

# [Rascunho] Esse blog é feito com JavaScript puro

<br>

*"E sinceramente não é algo que valha a pena se você estiver pensando em fazer a mesma coisa - a menos que seja algo mais estático."*

<br>

Vamos começar pelo começo, a ideia de criar um blog "modularizado", "dinâmico" e moderno com JavaScript puro parece - e é - algo
extremamente complexo. É possível chegar em um nível onde sua aplicação é completa, coerente e atende a todos esses requisitos,
mas é necessário um conhecimento muito mais completo sobre JavaScript, browsers e todo o contexto de front-end do que apenas o
básico - pelo menos se você quiser fazer algo "bom", ou minimamente "decente".

<br>

Eu particularmente não considero esse blog (falando a respeito do website, como uma peça de software) algo decente, ele é criado
a partir de experiências jogadas para o alto e organizadas de uma forma que me permitisse criar algo modularizado, mas isso está
longe de ser efetivo. Quando comecei a escrever o código desse blog, meu objetivo era "testar" como andavam os meus conhecimentos
sobre JavaScript e browsers, então me "desafiei" a escrever algo que funcionasse de uma forma parecida a um framework de frontend
moderno, com componentização e outras firulas. 

<br>

Acontece que chegar no estado da arte para essas funcionalidades não é uma função
simples. Não segui nenhuma especificação, não li o código-fonte de nenhum framework moderno de JavaScript, não fiz nada que não
fosse simplesmente olhar para uma aplicação moderna e tentar reproduzir o que ela fazia usando apenas meus conhecimentos, e esse
foi o motivo pelo qual esse blog se tornou uma peça de "frankestein" que seria inviável em uma aplicação comercial, porém me ajudou
a entender extensamente sobre algumas questões que eu nunca havia estudando antes, visto que na minha carreira eu majoritariamente
trabalhei com backend.

<br>

## Sumário

<br>

Dividi esse artigo em algumas partes, pois consegui extrair várias coisas extremamente importantes que eu julgo necessário uma pessoa que 
esteja começando o desenvolvimento de software - principalmente se tiver como objetivo atuar como frontend integralmente - aprender. 
Ou não, você também pode aprender por puro esporte e até mesmo simplesmente por aprender, de qualquer forma, recomendo
esse artigo a todos.

<br>

  - <a href='#componentização-e-rerendering'>Componentização e re-rendering</a>
    - <a href='#renderização-condicional'>Renderização condicional</a>
    - <a href='#sobre-propagação-de-eventos'>Sobre propagação de eventos</a>
    - <a href='#temas-e-cores'>Temas e cores</a>
    - <a href='#linguagens-e-localização'>Linguagens e localização</a>
  - <a href='#blogging-o-pacote-básico'>Blogging - o pacote básico</a>
    - <a href='#carregando-blogposts-de-arquivos-markdown'>Carregando blogposts de arquivos markdown</a>
    - <a href='#o-jeito-mais-incorreto-de-carregar-metadados'>O jeito mais incorreto de carregar metadados</a>
    - <a href='#pesquisando-entre-os-blogposts-carregados'>Pesquisando entre os blogposts carregados</a>
    - <a href='#seo-e-redirecionadores'>SEO e redirecionadores</a>
  - <a href='#code-highlighting-e-customização'>Code highlighting e customização</a>

<br>

<h2 id="componentização-e-rerendering">Componentização e re-rendering</h2>

<br>

Frameworks como React utilizam diversas técnicas para alcançar um nível de observabilidade que permite que o desenvolvedor final seja
capaz de criar aplicações componentizadas com re-rendering condicional baseado em um estado. No meu blog eu fiz algo diferente, e para
não falar que estou poluíndo o DOM com re-renders, fiz uma *gambiarra* com o intuito de salvar um estado prévio (o que, apesar de ser
um array aleatório sem muita preparação, podemos chamar de VirtualDOM).

<br>

<h3 id="renderização-condicional">Renderizaçao condicional</h3>

<br>

Nesse exemplo, faço uma funcionalidade que me permite emular renderização condicional na minha aplicação:

```js
class App {
    static currentPageContent = [];
    static pageContent = [];
}

function updatePageContent() {
  const currentTextContent = App.currentPageContent.join('');
  const newTextContent = App.pageContent.join('');

  if (currentTextContent != newTextContent) {
    // Pequeno hack para evitar reference traps
    App.currentPageContent = App.pageContent.map(x => x);
    document.body.innerHTML = newTextContent;
    document.body.style.backgroundColor = App.theme[App.theme.current].secondary;
  }
}
```

<br>

É nesse momento que eu já começo a me perguntar o que estou fazendo... Claro, a solução funciona, mas fica a pergunta para treinamento
mental: isso deveria realmente funcionar? Até o momento está funcionando maravilhosamente tanto no meu local quando no Github Pages, que
é onde eu hosteio a solução. Voltando para o contexto da situação, agora que já definimos a funcionalidade para atualizar nossa tela
automaticamente após atualização do nosso "VirtualDOM" (carinhosamente apelidado de *pageContent*), vamos definir um cronjob para que
seja feita essa checagem da maneira mais ~~limpa~~ possível.

<br>

```js
//Chamada inicial que busca evitar aquela tela vazia ao entrar no site
updatePageContent();

//Agora chamamos a cada 10 milissegundos porque realmente não nos preocupamos
//com as capacidades da máquina do usuário final.
setInterval(updatePageContent, 10);
```

<br>

Com tudo configurado, basta adicionar novos valores à variável *pageContent* e poderá ser vista a mágica acontecer: o DOM será
atualizado automaticamente. Tudo perfeito, correto? Agora tente colocar qualquer input de texto, estado ou número e veja a sua
aplicação explodir (não me responsabilizo por danos!). Falaremos disso mais à frente e o que podemos fazer para contornar esse
problema (tem tudo a ver com eventos e propagação! um ótimo exercício para JavaScript básico).

<br>

<h3 id="sobre-propagação-de-eventos">Sobre propagação de eventos</h3>

<br>

Eventos são seres traiçoeiros que vivem no JavaScript. Quando comecei a montar meu blog a partir desse sistema emendado de
renderização, logo percebi que havia um problema: não dava para interagir com inputs - eles iriam automaticamente dar
trigger no verificador de diferença de DOM e atualizar a tela, sempre que eu apertasse um botão. Então, ao invés de destruir
tudo que eu já fiz e fazer algo decente com React ou Vue, comecei a investigar os eventos do JavaScript e decidi implementar
o meu próprio leitor de eventos de input! (Só pra salvar estados, algo bem básico feito em cima dos eventos padrão do JavaScript).

<br>

Para começar, declarei o input - que por si só não causava uma renderização extra toda vez que era alterado, mas eu gostaria que a cada
caractere inserido, a função **loadPosts** que foi declarada da seguinte maneira (não vamos focar em código limpo nessa situação):

<br>

```js
async function loadPosts() {
    const index = await fetch('/posts/index.json')
        .then(data => data.json());

    for(const fileName of index) {
        const postText = await fetch(`/posts/${fileName}.md`)
            .then(data => data.text());

        const postItemData = postText
            .replaceAll('\r', '\n')
            .split("﹫﹫ ﹫﹫")
            .filter(post => post.trim() != "")
            .map(post => {
                const metadata = post
                    .trim()
                    .split('\n')[0]
                    .split('|')
                    .map(item => item.trim());

                const postContent = post
                    .trim()
                    .split('\n')
                    .slice(1)
                    .join('\n');

                return {
                    id: metadata[0],
                    title: metadata[1],
                    language: metadata[2],
                    date: new Date(metadata[3]).toLocaleDateString(),
                    description: metadata[4],
                    content: marked.parse(postContent)
                }
        });
        App.loadedPosts.push(...postItemData);
        App.loadedPosts.sort((a, b) => {
            return new Date(b.date) - new Date(a.date);
        });
    }
}
```
<br>

Essa função altera diretamente a variável global **App.loadedPosts**, o que faz com que novas renderizações incluam as postagens na lista visual fazendo com que haja uma alteração no código do DOM, o que faz com que o foco no input textual e o próprio valor sejam perdidos. Para evitar esse problema (ao menos na maioria dos browsers e talvez em alguns navegarores mobile) foi criada uma classe de estado, que será responsável por identificar os eventos de texto e orquestrar as alterações para fazer com que o texto seja devidamente salvo e o foco continue lá:

<br>

```js
const ViewerSearchState = {
    data: "",
    lastEvent: null,
    manage(event) {
        if (event.code.startsWith('Key') || event.code === "Backspace") {
            setTimeout(() => {
                ViewerSearchState.data = event.target.value;
                searchForPost(ViewerSearchState.data);
            }, 10);
            setTimeout(() => {
                const target = document.getElementById(event.target.id);
                target.focus();
                target.setSelectionRange(target.value.length, target.value.length);
            }, 20)
        }
    }
}
```

<br>

Nesse código é possível perceber que a funcionalidade manage cuida das execuções do input e são utilizados dois timeouts para fazer a obtenção dos elementos e suas devidas alterações/operações - os timeouts são necessários para "esperar" que os eventos do input sejam devidamente propagados, caso contrário poderão ocorrer bugs com valores vazios ou incompletos. Para chamar esse código, o seguinte HTML foi implementado no componente com **onkeydown**:

<br>

```html
<div style="margin-top: 10px">
    <input
        id="search"
        style="${style}" 
        type="text"
        value="${ViewerSearchState.data}"
        placeholder="${App.consts[App.consts.current].searchPlaceholder}"
        onkeydown="ViewerSearchState.manage(event, this)">
</div>
```

<br>

O resultado final foi um input text funcional que atende (na maioria dos casos) às necessidades de pesquisa do blog. Ainda existem alguns pontos a serem polidos principalmente no nível de compatibilidade entre os navegadores existentes e na efetividade da pesquisa, que está
ignorando alguns termos válidos.

<br>

<h3 id="temas-e-cores">Temas e cores</h3>

Criei um elemento de tema para dinamicamente alterar a tela com um clique de botão, como geralmente faço em outras bibliotecas e frameworks modernos, como React e Angular. Usando o localStorage, a aplicação consegue lembrar o estado do tema, para fazer essa sincronização eu utilizo apenas uma verificação de já existência dessa variável. Por enquanto, tenho dois temas, light e dark, com suas respectivas cores e configurações. O sistema de temas depende de alterar algumas propriedades do **document.body** e usar o **localStorage** para armazenar a preferência do usuário. 

<br>

Assim como qualquer outra coisa nessa aplicação, eu também utilizo uma variável no meu estado global para definir o tema atual, com o único diferencial de que eu faço essa sincronização extra simples para que eu possa persistir o dado e fazer com que ele fique guardado diretamente com a informação de domínio no navegador. Exemplo de código:

<br>

```js
static theme = {
    current: localStorage.getItem('theme-current') ?? 'dark',
    light: {
        primary: '#23272a',
        secondary: '#fffcfc',
        secondaryTransparent: 'rgba( 255, 255, 255, 0.6 )',
        borders: '#e8e9ed',
        primaryAlternative: '#5e6870',
        backgroundHover: '#f6f7f8'
    },
    dark: {
        primary: '#e8e9ed',
        secondary: '#23272a',
        secondaryTransparent: 'rgba( 0, 0, 0, 0.6 )',
        borders: '#fffcfc',
        primaryAlternative: '#5e6870',
        backgroundHover: '#404040'
    }
}
```

@@@@@
this-blog-was-made-with-pure-javascript | [Sketch] This blog was made with pure JavaScript | enus | 2024-09-21T18:14:57.044Z | Creating a modular, modern and dynamic blog with pure JavaScript, without TS or frameworks.

# [Sketch] This blog was made with pure JavaScript

Hello world