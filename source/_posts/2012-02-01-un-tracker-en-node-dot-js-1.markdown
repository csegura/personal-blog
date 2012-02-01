---
layout: post
title: "Un tracker en node.js (1)"
date: 2012-02-01 12:23
comments: true
categories: 
- nodejs
- mongodb

---
### Tarde ###

La verdad es que llevaba tiempo con ganas de hacer alguna cosita en node.js, pero no había tenido ocasión, así que con el remordimiento de llegar tarde he sacado un ratito para jugar un poco.

Como siempre el objetivo es aprender y evaluar. Para ello he pensado en hacer un tracker web, je otra cosa que parece que esta de moda ¿tendrá que ver?. 

La cosa es que he empezado y la cosa no pinta mal. Básicamente haré un tracker (luego mejorará), que será intrusivo, es decir que requerirá de un script en la página que nos enviará la información una vez esta pase al cliente. Vamos al igual que hace google analytics con su ga.js

Usaré [node.js][node], el framework de [express][express] para la front-end, [jade][jade] para las plantillas, [mongo][mongo] para almacenar los datos, [mongoose][mongoose] como módulo desde node.js

La información que guardaré por el momento la podemos ver en el modelo que he creado en mongo.

{% codeblock hosts.js lang:js %}
var mongoose = require('mongoose');

var Schema = mongoose.Schema
  , ObjectId = Schema.ObjectId;

var StatsSchema = new Schema({
  created_at: { type: Date, 'default': Date.now },
  tracked_at: Date,   // client datetime
  hostname:String,    // document.location.hostname
  port:Number,        // document.location.port
  protocol:String,    // document.location.protocol
  pathname:String,    // document.location.pathName
  referrer:String,    // document.referrer (r)
  title:String,       // document.title
  user_agent:String,  // navigator usr agent
  address:String,     // end-client address (ip)
  lng:Number,         // geo-code
  lat:Number
});

var HostsSchema = new Schema({
  code:Number,
  host:String,
  stats:[StatsSchema]
});

var exports = module.exports = Hosts = mongoose.model('Hosts', HostsSchema);
{% endcodeblock %}


desde nuestra app enviaremos el script, para ello he añadido dos rutas

{% codeblock lang:js %}
app.get('/wstat.js', function (req, res) {
  res.sendfile(__dirname + '/public/javascripts/wstat.js');
});

app.get('/wstat.gif', function (req, res) {
  saveStats(req);
  res.sendfile(__dirname + '/public/images/wstat.gif');
});
{% endcodeblock %}

el cliente primero demandará nuestro script wstat.js, mediante el siguiente script que se cargará de manera asíncrona, es básicamente lo que hace google analytics genera el DOM y lo inserta en nuestra página, incluso lleva nuestro código de seguimiento para poder rastrear diferentes sitios 

{% codeblock wstat_client.js lang:js %}
(function() {
  var wstat = document.createElement('script');
  wstat.type = 'text/javascript';
  wstat.async = true;
  wstat.id = 'wstat';
  wstat.setAttribute('code', '9999');
  wstat.src = '//HOST_NAME/wstat.js';
  var e = document.getElementsByTagName('script')[0];
  e.parentNode.insertBefore(wstat, e);
})();
{% endcodeblock %}

este script lo que hace es cargar wstat.js que será el encargado de trasmitirnos parte de la información relevante a nuestra página, la otra parte la extraeremos de los datos de la petición (request)

{% codeblock wstat.js lang:js %}
(function() {
  var script = document.getElementById('wstat');
  var code = script.getAttribute('code');
  var image = new Image();
  var url = script.src.replace('/wstat.js', '/wstat.gif');
  url += "?c=" + encodeURIComponent(code);
  url += "&d=" + new Date().getTime();
  url += "&h=" + encodeURIComponent(document.location.hostname);
  url += "&p=" + encodeURIComponent(document.location.pathname);
  url += "&pl=" + encodeURIComponent(document.location.protocol);
  url += "&pr=" + encodeURIComponent(document.location.port);
  url += "&rf=" + encodeURIComponent(document.referrer);
  url += "&t=" + encodeURIComponent(document.title);
  image.src = url;
})();
{% endcodeblock %}

este script, genera en el dom una imagen y como src le pasa la dirección de nuestro script sustituyendo el script por una imagen y añadiendo los parámetros que queremos obtener del cliente.

cuando se solicita la imagen hay un **saveStats(req)** que es el que se encarga de almacenar los datos en nuestra base de datos

{% codeblock lang:js %}
function saveStats(req) {	
    var code = req.param('c', null);
    if (code) {
        var stats = getStats(req);
        Hosts.findOne({ code: code}, function (err, doc) {
            if (!doc) {
                doc = new Hosts({
                    code:code,
                    stats: []
                });
                doc.save();
            }
            doc.stats.push(stats);
            doc.save();
        });
    }
}
{% endcodeblock %}

primero busca si tenemos ya una entrada para nuestro código de seguimiento, en caso de que no exista la crea y almacena las estadísticas que obtenemos de la petición

{% codeblock lang:js %}
function getStats(req) {
  var date = req.param('d');
  var hostname = decodeURIComponent(req.param('h'));
  var pathname = decodeURIComponent(req.param('p'));
  var protocol = decodeURIComponent(req.param('pl'));
  var port = decodeURIComponent(req.param('pr'));
  var referrer = decodeURIComponent(req.param('rf'));
  var title = decodeURIComponent(req.param('t'));

  return {
    tracked_at: date,
    hostname: hostname,
    pathname: pathname,
    protocol: protocol,
    port: port,
    referrer: referrer,
    user_agent:req.headers['user-agent'],
    address: req.socket.remoteAddress,
    title: title
  };
}
{% endcodeblock %}

#### primeras conclusiones ####

Bueno, la programación de este mini proyecto solo me ha tomado unas horas, lo cual es BUENO, como prototipo la cosa ha ido bastante decente, alguna pequeña cosa a la hora de hacer el deployment que por el momento hago un un shell script .. de las pruebas y de otras cosas hablaré en otros posts, que esto va para largo y de momento node.js se queda.


[node]: http://nodejs.org
[express]: http://expressjs.com/
[jade]: http://jade-lang.com/
[mongo]: http://www.mongodb.org/
[mongoose]: http://mongoosejs.com/
 