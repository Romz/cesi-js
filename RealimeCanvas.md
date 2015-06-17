RealtimeCanvas
==============

Principe
--------
Faire une application de dessins collaboratif à l'aide d'un canvas HTML5 et des websockets

Technos
-------
- Nodejs
- Express
- Socket.io
- HTML5 canvas
- Twitter bootstrap
- jQuery
- Bower


Instalation et création du projet
---------------------------------
Installer nodejs en allant sur ce lien: [nodejs](https://nodejs.org/)

Installer bower
```
npm install -g bower
```

Créer le répertoire du projet (par exemple:  RealtimeCanvas).

Créer un fichier package.json dans ce repertoire :
```json
{
  "name": "RealtimeCanvas",
  "version": "0.0.1",
  "description": "",
  "dependencies": {
  }
}
```

Installer express dans le projet.

```
npm install express --save
```
Installer socket.io via npm.

```
npm install socket.io --save
```

Initialiser bower (Génère le fichier bower.json qui gère les dépendances des différentes librairies du projet).
```
bower init
```

Ajouter jquery.
```
bower install jquery --save
```

Ajouter Twitter bootstrap.
```
bower install bootstrap --save
```

Les librairies téléchargé via bower se trouve dans le répertoire "bower_components".


Creation du server web
----------------------

Maintenant que toutes les installations sont faites, nous allons créer le serveur qui va permettre de faire tourner notre application.

Créer un fichier app.js à la racine du répertoire de votre application. C'est ce fichier qui va permettre de lancer le serveur http:

Nous allons dans un premier temps initialiser notre serveur:

```js
var express = require('express'); // Recupération de la librairie express
var app = express(); // Initialisation d'express
var server = require('http').Server(app); // Création du serveur http
var io = require('socket.io').listen(server); // Initialisation de socket.io

server.listen(3000); // Lancement du server sur le port 3000 (Ou un autre port si le 3000 est déjà pris)
```

Pour lancer le serveur, utiliser la commande :

```
node app.js
```
À chaque modification du fichier app.js, il faut redémarer le serveur.

Maintenant, si on essaye d'accéder au server (via http://localhost:3000), on devrait voir apparaître le message suivant: "Cannot GET /".
Cela indique que le serveur est bien lancé, mais qu'aucune route n'est définie.

Pour éteindre le serveur, utilisez le raccourcis Ctrl + C.

Nous allons donc créer une route qui va pointer vers un fichier index.html:

```js
app.get('/', function (req, res) { 
  res.sendFile(__dirname + '/index.html');
});
```

Notre application va aussi utiliser un certain nombre d'assets, comme nos fichiers js et css et les différentes librairies du projet.
Nous allons donc rendre certain répertoire disponible pour pouvoir lire ces fichier statiques depuis le navigateur:

Il faut donc maintenant créer un fichier index.html qui contiendra la page de notre application:
```js
app.use('/assets', express.static('assets')); 
app.use('/bower_components', express.static('bower_components'));
```

Nous avons donc rendu accéssible le répertoire assets qui va contenir nos fichiers js et css, ainsi que le répertoire bower_components pour toutes les différentes librairies.

Nous allons maintenant créer le fichier index.html qui va contenir notre frontend:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Realtime canvas</title>
    <!-- The stylesheets -->
    <link rel="stylesheet" href="assets/css/styles.css" /><!-- Notre feuille de style -->
    <link rel="stylesheet" href="bower_components/bootstrap/dist/css/bootstrap.min.css" /><!-- Le css bootstrap -->
  </head>
  <body>
    <div id="main">
	  <div class="container">
	    <h1>Realtime Canvas</h1>
		<div class="row">
		  <div class="col-sm-9 bar-left">
		    <div id="cursors"></div>
		    <canvas id="board" width="1900" height="1000"><!-- La balise canvas est une balise HTML5 qui permet de dessiner des figure via le javascript. C'est une sorte de feuille blanche qui va contenire notre dessin.-->
HTML5 Canvas not supported
            </canvas>
          </div>
          <div class="col-sm-3 bar-right">
            Options goes here
          </div>
        </div>
      </div>
    </div>
	  
    <script src="/socket.io/socket.io.js"></script>
    <script src="bower_components/jquery/dist/jquery.min.js"></script>
    <script src="assets/js/script.js"></script>
  </body>
</html>
```

Nous utilisons ici le système de grille de bootstrap qui est exliqué ici: [Grid bootstrap](http://getbootstrap.com/css/#grid).
Voilà, nous avons la base HTML de notre application.

Maintenant, nous allons créer les assets manquant:
Dans le répertoire du projets, créer l'arborescence suivante:

- assets
  - css
    - style.css (feuille de style de l'application)
  - js
    - script.js (fichier js qui va contenir le code de l'application)


Dessin
------

Maintenant que le serveur est près et que tous les fichiers sont prèt, nous allons pouvoir entre dans le vif du sujet.
Nous allons ici faire en sorte de dessiner sur notre canvas.

Dans un premier temps, nous allons ajouter un peu de css dans notre feuille de style pour distinguer notre canvas:

```css
#board {
  background: #eee;
}
```

Le canvas doit maintenant être gris claire.

Nous allons maintenant aller dans le fichier script.js pour écrire le code de l'application:

```js
$(document).ready(function(){
  // Code goes here
});
```

Nous utilisons ici jQuery pour pouvoir manipuler plus facilement le DOM et les événements.
Le $ fait référence à l'objet jQuery. On lui passe le document en paramètre et on appel la fonction ready qui est appelé lorsque le document HTML est chargé.
Tout le js que nous allons écrire se trouvera dans cette fonction.

Maintenant que jQuery est initialisé, nous allons afficher un message pour alerter les utilisateurs qui aurait un navigateur ne supportant pas les canvas HTML5:

```js
if(!('getContext' in document.createElement('canvas'))){
  alert('Your browser does not support canvas!');
  return false;
}
```

Nous allons ensuite initialiser un certain nombre de variables qui seront utilie par la suite

```js
var win = $(window),              // Récupération de l'élément window
doc = $(document),                // Récupération de l'élément document
canvas = $('#board'),             // Récupération de l'élément canvas
offset = canvas.offset(),         // Offset du canvas (Décalage par rapport à la page)
ctx = canvas[0].getContext('2d'), // Contexte 2D du canvas pour le dessin
drawing = false,                  // Booléen permettant de savoir si on est en train de dessiner
mouse = {};                       // Position de la souris
```

Nous allons maintenant nous attacher à l'événement de la souris 'mousedown', qui est l'événement qui est lancé quand on appuis sur le bouton de la souris:

```js
canvas.on('mousedown',function(e){
  // Code on mousedown event goes here
});
```

Dans cette fonction nous allons stoquer les coordonnées de la souris, et commencer le dessin:

```js
e.preventDefault(); // Permet d'empécher le comportement par défault de l'élément
drawing = true; // On passe drawing à true car nous sommes en train de dessiner

mouse.x = e.pageX - offset.left; // On stock l'abscisse relative au canvas
mouse.y = e.pageY - offset.top; // On stock l'ordonnée relative au canvas
```

Nous allons maintenant nous attacher à l'événement de la souris 'mousemove', qui est l'événement qui est lancé à chaque fois que la souris bouge:

```js
canvas.on('mousemove',function(e){
  // Code on mousemove event goes here
});
```

Dans cette fonction nous allons récupérer les coordonnées de la souris, et dessiner dans le canvas si la variable drawing est à true:

```js
var x = e.pageX - offset.left;
var y = e.pageY - offset.top;

if(drawing){ // On ne dessine que si on est en train de cliquer dans le canvas: 
  drawLine(mouse.x, mouse.y, x, y, {}); // Fonction de dessin que l'on va définir après
  // On stoque les nouvelles coordonnées de la souris
  mouse.x = x; 
  mouse.y = y;
}
```

Nous allons maintenant nous attacher à l'événement de la souris 'mouseup', qui est l'événement qui est lancé quand on relache le bouton de la souris, ainsi que l'événement mouseleave qui est lancé quand la souris sort de l'élément :

```js
canvas.on('mouseup mouseleave', function(e){
  drawing = false; // si on relache le bouton de la souris ou qu'on sort du cadre, on arrête de dessiner
});
```

Il ne nous reste plus maintenant qu'à faire la fonction qui permet de dessiner:

```js
function drawLine(
  fromx, // coordonnée x de départ
  fromy, // coordonnée y de départ
  tox,   // coordonnée x d'arrivé
  toy,   // coordonnée y d'arrivé
  options){ // des options (Plus tard dans le TP)
  ctx.lineCap = 'round'; // Sert a arrondir le tracé pour qu'il soit plus fluide
  ctx.beginPath(); // Permet de commencer un nouveau chemin
  ctx.moveTo(fromx, fromy); // Place le curseur aux anciennes coordonnées
  ctx.lineTo(tox, toy); // Crée un ligne entre le curseur courant (ici les derniéres coordonnées de la souris) et les coordonnées passé en paramètre (ici les nouvelles coordonnées de la souris).
  ctx.stroke(); // Dessine le chemin que l'on vient de définir
  ctx.closePath(); // Termine le chemin 
}
```

Voilà, vous pouvez à présent dessiner sur le canvas avec la souris.

Options
-------

Nous allons maintenant ajouter quelques options, comme par exemple la possibilité de changer la couleur ou la taille du crayon.
Pour cela, nous allons rajouter une variable au niveau des initialisations :

```js
var options = { // Options de dessin
  color: "#000000",  // Couleur en noir par défault
  size: 1           // Taille de 1 par défault
};
```

Nous allons maintenant rajouter les options dans la fonction de dessin drawLine, ce qui donne:
```js
function drawLine(
  fromx, // coordonnée x de départ
  fromy, // coordonnée y de départ
  tox,   // coordonnée x d'arrivé
  toy,   // coordonnée y d'arrivé
  options){ // des options (Plus tard dans le TP)
  ctx.lineCap = 'round'; // Sert a arrondir le tracé pour qu'il soit plus fluide
  ctx.beginPath(); // Permet de commencer un nouveau chemin
  ctx.moveTo(fromx, fromy); // Place le curseur aux anciennes coordonnées
  ctx.lineTo(tox, toy); // Crée un ligne entre le curseur courant (ici les derniéres coordonnées de la souris) et les coordonnées passé en paramètre (ici les nouvelles coordonnées de la souris).
  
  ctx.strokeStyle = options.color; // on assigne la couleur du trait
  ctx.lineWidth = options.size; // On assigne la taille du trait
  
  ctx.stroke(); // Dessine le chemin que l'on vient de définir
  ctx.closePath(); // Termine le chemin 
}
```

Il faut maitenant ajouter les options à l'appel de la fonction drawLine dans l'événement mousemove :

```js
drawLine(mouse.x, mouse.y, x, y, options);
```

On peut maintenant changer la couleur et la taille du trait en changeant les valeurs dans la variable option.

Nous allons maintenant laisser la possibilité à l'utilisateur de changer la couleur et la taille de son crayon.

Pour cela, nous allons ajouter des champs dans la barre de droite de notre page (div avec la classe bar-right dans index.html):

```html
<div class="options">
  <div>
   <label>Couleur:</label>
   <input type="color" id="color" class="options"/> <!-- Champs HTML qui permet d'avoir un colorpicker-->
  </div>
  <div>
    <label>Taille:</label>
    <input type="number" value="1" id="size" class="options" width="2"/> <!-- Champs number pour la taille -->
  </div>
</div>
```

Il nous faut maintenant changer les valeurs de la variable options par les valeura des champs. Pour cela, nous allons nous relier à l'événement change de ces éléments:

```js
$(".options").on('change', function(){ // Fonction exécuté lors de l'événement change de tous les éléments ayant la classe options
  options[$(this).attr('id')] = $(this).val(); // On récupére l'id de l'élément et on s'en sert comme index pour l'objet options en lui affectant à valeur
});
```

On peut maintenant choisir la couleur et la taille du crayon.

Temps réel
----------

Maintenant que nous pouvont dessiner sur notre beau tableau, nous allons ajouter la couche réseau pour pouvoir dessiner à plusieurs en temps réel.
Le principe est simple. Quand un utilisateur bouge la souris, on envoie au serveur de socket les coordonnées de la souris et les options, et le serveur lui envois le tout à tous les autres utilisateurs.

Pour cela nous allons retourner dans le fichier app.js pour ajouter la partie socket.io.
Dans un premier temps, il faut se lier à l'événement connections des sockets :

```js
io.sockets.on('connection', function (socket) {
 // Code exécuté quand un utilisateur se connecte sur le serveur
 // l'objet socket passé en paramètre est l'objet qui sert à gérer la connexion avec l'utilisateur
});
```

Dans cet fonction, nous allons écouter l'événement "drawing" (événement custom), récupérer les informations utilisateurs et les renvoyer à tout le monde:

```js
socket.on('drawing', function(data) {
  var result = { 
    data:data, // Données envoyé par les utilisateurs
    id: socket.id // Identifiant unique du socket généré automatiquement
  }
  socket.broadcast.emit('drawing', result); // Envoi à tout le monde des données
});
```

Dans le client (script.js), nous allons maintenant ajouter la connexion au serveur de socket.
Dans la partie initialisation des variables, ajouter :

```js
var socket = io.connect(''); // Connexion au serveur de socket
var clients = {}; // List des clients connectées
var cursors = {}; // Curseurs de clients connectées
var name = "NoName"; // Le nom de l'utilisateur
```

Le paremètre de la fonction connect est l'url du serveur de socket. Si on ne met rien, c'est le domaine courrant qui est choisi.

Nous allons maintenant envoyer les coordonnées et options du mouvement de la souris au serveur de socket.
Dans l'événement mousemove, ajouter ceci:

```js
socket.emit('drawing', { // La fonction emit permet d'envoyer un événement au serveur. Le premier paramètre est le nom de l'événement, et les deuxième les données à envoyer.
  x: x,
  y: y,
  drawing: drawing,
  options: options,
  name:name
});
```

Cela va permettre d'envoyer au serveur toutes les informations.

Il faut maintenant récupérer les informations des autres utilisateurs.

Pour cela, nous allons nous attaché à l'événement 'drawing' du serveur de socket:

```js
socket.on("drawing", function(data) {
  // Récupération des données des autres utilisateurs
});
```

Nous allons maintenant dessiner dans notre canvas ce que dessine les autres: 

```js
if(data.data.drawing && clients[data.id]) { // Si le client existe et qu'il est en train de dessiner
  var c = clients[data.id];
  drawLine(c.x, c.y, data.data.x, data.data.y, c.options); 
}
clients[data.id] = data.data; // on stock les dernières informations des utilisateurs
```

Vous pouvez maintenant essayer de vous connecter à plusieurs navigateurs.

Nous allons maintenant afficher la position de la souris des autres utilisateurs.

Dans l'événement drawing, rajouter le code suivant:

```js
if(!(data.id in clients)) { // Si le client existe
  var cursor = $('<div class="cursor">').appendTo('#cursors'); // On creer un élément curseur
  cursor.text(data.data.name); // On ajoute le nom de l'utilisateur
  cursors[data.id] = cursor;  // On stock l'élément dans une variable
}

cursors[data.id].css({ // Mise à jour de la position du cursor 
  'left' : data.data.x + 15,
  'top' : data.data.y
});
```

Nous allons rajouter le style du curseur:

```css
.cursor {
  position: absolute;
  border: 1px solid black;
}
```

Page de connexion
-----------------

Nous allons maintenant faire une page de connexion pour choisir son nom d'utilisateur.

Dans le fichier index.html, ajouter le formulaire de connexion au début du body:

```html
<div id="connection" class="popup">
  <div class="inner">
    <h2>Connection</h2>
    <form id="connection-form">
      <label>Nom:</label>
      <input id="connection-name" type="text" />
      <input type="submit" value="Connexion"/>
    </form>
  </div>
</div>
```

Dans le fichier style.css, nous allons ajouter le style de la popup:

```css
.popup {
  position: fixed;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.7);
  top: 0;
  left:0;
  z-index: 1;
  opacity: 1;
  visibility: visible;
  transition: opacity 1s linear;
}

.popup.connection-hidden {
  opacity: 0;
  visibility: hidden;
  transition: visibility 0s 1s,  opacity 1s linear;
}

.popup .inner {
  position:absolute;
  top: 50%;
  left: 50%;
  background: white;
  padding: 20px;
  transform: translate(-50%, -50%);
}
```

Nous allons maintenant ajouter le code pour stoquer le nom d'utilisateur.

```js
$('#connection-form').on('submit', function(e){ // On s'attache sur l'événement submit du formulaire
  e.preventDefault();  // On empèche le comportement par défault du formulaire
  $('#connection').addClass('connection-hidden'); // On ajoute la classe pour cacher le formulaire
  var val = $("#connection-name").val(); // On récupère la valeur du formulaire
  if(val != "") {
    name = val; // On stoque le nom de l'utilisateur
  }
});
```

Chat
----
Nous allons maintenant ajouter un chat dans la barre de droite.
Nous allons donc ajouter l'html nécessaire dans le fichier index.html dans la barre de droite:

```html
<div class="chat">
  <div id="messages">
  </div>
  <form id="messages-form">
   <input type="text" placeholder="Message" id="message"/>
  </form>
</div>
```

Dans le fichier style.css ajouter les styles suivant: :

```css
#messages {
  height: 400px;
  overflow: auto;
  padding: 5px;
  border: 1px solid black;
}
```

Nous allons maintenant envoyer le message au serveur :

```js
$("#messages-form").on('submit', function(e){ 
  e.preventDefault();
  var message = $('#message').val(); // On récupère le message du champs texte
  socket.emit('message', message); // On envoi le message au serveur
  postMessage(message, name);  // On affiche le message dans la div messages
  $('#message').val(''); // On vide le champs message
});
```

Nous allons maintenant récupérer le message sur le serveur pour le renvoyer à tout le monde.
Dans le fichier app.js, ajouter le code suivant:

```js
socket.on('message', function(message){
  socket.broadcast.emit('message', {
    id: socket.id,
    message: message
  });
});
```

Nous allons maintenant récupérer les messages des autres utilisateurs dans le client.
Pour cela, nous allons nous attacher à l'événement 'message' du serveur.
Dans le fichier script.js, ajouter :

```js
socket.on('message', function(data) {
  var c = clients[data.id];
  postMessage(data.message, c.name);
});
```

Il ne reste plus qu'à créer la fonction postMessage qui permet d'afficher les messages dans la div message:

```js
function postMessage(message, name) {
  var messageWrapper = $('<div class="message"></div>'); // On crée l'élément message
  var nameElement = $('<div class="name"></div>').text(name); // On ajoute l'élément contenant le nom de l'utilisateur
  var messageElement = $('<div class="message"></div>').text(message); // On ajoute l'élément contenant le message
  messageWrapper.append(nameElement).append(messageElement).appendTo("#messages"); // On ajoute tous les éléments à la div messages
  var elem = $('#messages')[0]; // On récupére l'élément messages
  elem.scrollTop = elem.scrollHeight; // On règle le scroll pour qu'il soit tout en bas à chaque message
}
```

Bonus
-----
- Ajouter des formes comme des réctangles ou des cercle
- Ajouter la possibilité de mettre du texte
- Sauvegarder l'historique du chat
- Sauvegarder le dessin
- Ajouter des commandes dans le chats pour intéragir avec le dessin
- Ajouter une couleur différentes à chaque utilisateur (Curseur et chat)
- Streamer la webcam


Références
----------
- [Nodejs](https://nodejs.org/)
- [socket.io](http://socket.io/)
- [Express](http://expressjs.com/)
- [jQuery](https://jquery.com/)
- [Twitter bootstrap](http://getbootstrap.com/)
- [Bower](http://bower.io/)
