libs:
  - d3
  - domtree

---

# Árvore DOM

A espinha dorsal de um documento HTML são as tags.

De acordo com o Document Object Model (DOM), cada tag HTML é um objeto. As tags aninhadas são “filhas” da que a contém. O texto dentro de uma tag também é um objeto.

Todos esses objetos são acessíveis usando JavaScript, e podemos usá-los para modificar a página.

Por exemplo, `document.body` é o objeto que representa a tag `<body>`.

Executar este código deixará `<body>` vermelho por 3 segundos:

```js run
document.body.style.background = 'red'; // torna o fundo vermelho

setTimeout(() => document.body.style.background = '', 3000); // retornar ao anterior
```

Aqui usamos `style.background` para alterar a cor de fundo de `document.body`, mas existem muitas outras propriedades, como:

- `innerHTML` -- conteúdo HTML do nó.
- `offsetWidth` -- a largura do nó (em pixels)
- ...e assim por diante.

Em breve aprenderemos mais maneiras de manipular o DOM, mas primeiro precisamos saber sobre sua estrutura.

## Um exemplo do DOM

Vamos começar com o seguinte documento simples:
sgsdf
```html run no-beautify
<!DOCTYPE HTML>
<html>
<head>
  <title>About elk</title>
</head>
<body>
  The truth about elk.
</body>
</html>
```

O DOM representa o HTML como uma estrutura de árvore de tags. É assim que ele se parece:

<div class="domtree"></div>

<script>
let node1 = {"name":"HTML","nodeType":1,"children":[{"name":"HEAD","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"\n  "},{"name":"TITLE","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"About elk"}]},{"name":"#text","nodeType":3,"content":"\n"}]},{"name":"#text","nodeType":3,"content":"\n"},{"name":"BODY","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"\n  The truth about elk.\n\n\n"}]}]}

drawHtmlTree(node1, 'div.domtree', 690, 320);
</script>

```online
Na imagem acima, você pode clicar nos nós dos elementos e seus filhos irão abrir/recolher.
```

Cada nó da árvore é um objeto.

Tags são *nós de elemento* (ou apenas elementos) e formam a estrutura da árvore: `<html>` está na raiz, então `<head>` e `<body>` são seus filhos, etc.

O texto dentro dos elementos forma *nós de texto*, rotulados como `#text`. Um nó de texto contém apenas uma string. Ele pode não ter filhos e é sempre uma folha da árvore.

Por exemplo, a tag `<title>` tem o texto `"About elk"`.

Observe os caracteres especiais nos nós de texto:

- uma nova linha: `↵` (conhecido em JavaScript como `\n`)
- um espaço: `␣`

Espaços e novas linhas são caracteres totalmente válidos, como letras e dígitos. Eles formam nós de texto e se tornam parte do DOM. Portanto, por exemplo, no exemplo acima, a tag `<head>` contém alguns espaços antes de `<title>`, e esse texto se torna um nó `#text` (ele contém uma nova linha e alguns espaços apenas).

Existem apenas duas exclusões de nível superior:
1. Espaços e novas linhas antes de `<head>` são ignorados por motivos históricos.
2. Se colocarmos algo depois de `</body>`, esse algo será automaticamente movido para dentro do `body`, no final, pois a especificação HTML exige que todo o conteúdo esteja dentro de `<body>`. Portanto, não pode haver espaços depois de `</body>`.

Em outros casos, tudo é simples -- se houver espaços (como qualquer caractere) no documento, eles se tornam nós de texto no DOM e, se removê-los, não haverá mais nenhum.

Aqui não há nós de texto apenas com espaço:

```html no-beautify
<!DOCTYPE HTML>
<html><head><title>About elk</title></head><body>The truth about elk.</body></html>
```

<div class="domtree"></div>

<script>
let node2 = {"name":"HTML","nodeType":1,"children":[{"name":"HEAD","nodeType":1,"children":[{"name":"TITLE","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"About elk"}]}]},{"name":"BODY","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"The truth about elk."}]}]}

drawHtmlTree(node2, 'div.domtree', 690, 210);
</script>

```smart header="Os espaços nas bordas e o texto vazio intermediário geralmente estão ocultos nas ferramentas"
Ferramentas de navegador (a serem abordadas em breve) que trabalham com DOM geralmente não mostram espaços no início/fim do texto e nós de texto vazios (quebras de linha) entre as tags.

Isso porque eles são usados principalmente para decorar HTML e não afetam a forma como ele é mostrado (na maioria dos casos).

Em outras imagens da DOM, às vezes as omitimos onde são irrelevantes, para manter as coisas curtas.
```


## Autocorreção

Se o navegador encontrar HTML malformado, ele o corrigirá automaticamente ao criar o DOM.

Por exemplo, a tag superior é sempre `<html>`. Mesmo que não exista no documento, ela existirá no DOM, porque o navegador o criará. O mesmo vale para `<body>`.

Como exemplo, se o arquivo HTML é uma única palavra `"Hello"`, o navegador irá envolvê-lo em `<html>` e `<body>` e adicionar o `<head>` necessário, então o DOM será:


<div class="domtree"></div>

<script>
let node3 = {"name":"HTML","nodeType":1,"children":[{"name":"HEAD","nodeType":1,"children":[]},{"name":"BODY","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"Hello"}]}]}

drawHtmlTree(node3, 'div.domtree', 690, 150);
</script>

Ao gerar o DOM, os navegadores processam automaticamente os erros no documento, fecham as tags e assim por diante.

Um documento com tags não fechadas:

```html no-beautify
<p>Hello
<li>Mom
<li>and
<li>Dad
```

...se tornará um DOM normal conforme o navegador lê as tags e restaura as partes ausentes:

<div class="domtree"></div>

<script>
let node4 = {"name":"HTML","nodeType":1,"children":[{"name":"HEAD","nodeType":1,"children":[]},{"name":"BODY","nodeType":1,"children":[{"name":"P","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"Hello"}]},{"name":"LI","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"Mom"}]},{"name":"LI","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"and"}]},{"name":"LI","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"Dad"}]}]}]}

drawHtmlTree(node4, 'div.domtree', 690, 360);
</script>

````warn header="Tabelas sempre têm `<tbody>`"
An interesting "special case" is tables. By DOM specification they must have `<tbody>` tag, but HTML text may omit it. Then the browser creates `<tbody>` in the DOM automatically.

For the HTML:

```html no-beautify
<table id="table"><tr><td>1</td></tr></table>
```

DOM-structure will be:
<div class="domtree"></div>

<script>
let node5 = {"name":"TABLE","nodeType":1,"children":[{"name":"TBODY","nodeType":1,"children":[{"name":"TR","nodeType":1,"children":[{"name":"TD","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"1"}]}]}]}]};

drawHtmlTree(node5,  'div.domtree', 600, 200);
</script>

You see? The `<tbody>` appeared out of nowhere. We should keep this in mind while working with tables to avoid surprises.
````

## Other node types

Let's add more tags and a comment to the page:

```html
<!DOCTYPE HTML>
<html>
<body>
  The truth about elk.
  <ol>
    <li>An elk is a smart</li>
*!*
    <!-- comment -->
*/!*
    <li>...and cunning animal!</li>
  </ol>
</body>
</html>
```

<div class="domtree"></div>

<script>
let node6 = {"name":"HTML","nodeType":1,"children":[{"name":"HEAD","nodeType":1,"children":[]},{"name":"BODY","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"\n  The truth about elk.\n  "},{"name":"OL","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"\n    "},{"name":"LI","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"An elk is a smart"}]},{"name":"#text","nodeType":3,"content":"\n    "},{"name":"#comment","nodeType":8,"content":"comment"},{"name":"#text","nodeType":3,"content":"\n    "},{"name":"LI","nodeType":1,"children":[{"name":"#text","nodeType":3,"content":"...and cunning animal!"}]},{"name":"#text","nodeType":3,"content":"\n  "}]},{"name":"#text","nodeType":3,"content":"\n\n\n"}]}]};

drawHtmlTree(node6, 'div.domtree', 690, 500);
</script>

Here we see a new tree node type -- *comment node*, labeled as `#comment`.

We may think -- why is a comment added to the DOM? It doesn't affect the visual representation in any way. But there's a rule -- if something's in HTML, then it also must be in the DOM tree.

**Everything in HTML, even comments, becomes a part of the DOM.**

Even the `<!DOCTYPE...>` directive at the very beginning of HTML is also a DOM node. It's in the DOM tree right before `<html>`. Few people know about that. We are not going to touch that node, we even don't draw it on diagrams, but it's there.

The `document` object that represents the whole document is, formally, a DOM node as well.

There are [12 node types](https://dom.spec.whatwg.org/#node). In practice we usually work with 4 of them:

1. `document` -- the "entry point" into DOM.
2. element nodes -- HTML-tags, the tree building blocks.
3. text nodes -- contain text.
4. comments -- sometimes we can put information there, it won't be shown, but JS can read it from the DOM.

## See it for yourself

To see the DOM structure in real-time, try [Live DOM Viewer](http://software.hixie.ch/utilities/js/live-dom-viewer/). Just type in the document, and it will show up as a DOM at an instant.

## In the browser inspector

Another way to explore the DOM is to use the browser developer tools. Actually, that's what we use when developing.

To do so, open the web page [elk.html](elk.html), turn on the browser developer tools and switch to the Elements tab.

It should look like this:

![](elk.svg)

You can see the DOM, click on elements, see their details and so on.

Please note that the DOM structure in developer tools is simplified. Text nodes are shown just as text. And there are no "blank" (space only) text nodes at all. That's fine, because most of the time we are interested in element nodes.

Clicking the <span class="devtools" style="background-position:-328px -124px"></span> button in the left-upper corner allows us to choose a node from the webpage using a mouse (or other pointer devices) and "inspect" it (scroll to it in the Elements tab). This works great when we have a huge HTML page (and corresponding huge DOM) and would like to see the place of a particular element in it.

Another way to do it would be just right-clicking on a webpage and selecting "Inspect" in the context menu.

![](inspect.svg)

At the right part of the tools there are the following subtabs:
- **Styles** -- we can see CSS applied to the current element rule by rule, including built-in rules (gray). Almost everything can be edited in-place, including the dimensions/margins/paddings of the box below.
- **Computed** -- to see CSS applied to the element by property: for each property we can see a rule that gives it (including CSS inheritance and such).
- **Event Listeners** -- to see event listeners attached to DOM elements (we'll cover them in the next part of the tutorial).
- ...and so on.

The best way to study them is to click around. Most values are editable in-place.

## Interaction with console

As we explore the DOM, we also may want to apply JavaScript to it. Like: get a node and run some code to modify it, to see how it looks. Here are few tips to travel between the Elements tab and the console.

- Select the first `<li>` in the Elements tab.
- Press `key:Esc` -- it will open console right below the Elements tab.

Now the last selected element is available as `$0`, the previously selected is `$1` etc.

We can run commands on them. For instance, `$0.style.background = 'red'` makes the selected list item red, like this:

![](domconsole0.svg)

From the other side, if we're in console and have a variable referencing a DOM node, then we can use the command `inspect(node)` to see it in the Elements pane.

Or we can just output the DOM node in the console and explore "in-place", like `document.body` below:

![](domconsole1.svg)

That's for debugging purposes of course. From the next chapter on we'll access and modify DOM using JavaScript.

The browser developer tools are a great help in development: we can explore the DOM, try things and see what goes wrong.

## Summary

An HTML/XML document is represented inside the browser as the DOM tree.

- Tags become element nodes and form the structure.
- Text becomes text nodes.
- ...etc, everything in HTML has its place in DOM, even comments.

We can use developer tools to inspect DOM and modify it manually.

Here we covered the basics, the most used and important actions to start with. There's an extensive documentation about Chrome Developer Tools at <https://developers.google.com/web/tools/chrome-devtools>. The best way to learn the tools is to click here and there, read menus: most options are obvious. Later, when you know them in general, read the docs and pick up the rest.

DOM nodes have properties and methods that allow us to travel between them, modify them, move around the page, and more. We'll get down to them in the next chapters.
