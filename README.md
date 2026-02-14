<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>TERMINAL POÉTICA v8.2</title>
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap" rel="stylesheet">
<style>
body{background:black;color:#00ff41;font-family:'Share Tech Mono', monospace;font-size:22px;margin:0;overflow:hidden;}
#monitor-frame{position:fixed;inset:5%;border-radius:22px;border:2px solid rgba(200,255,255,0.7);
box-shadow:0 0 40px rgba(0,255,255,0.35), inset 0 0 20px rgba(255,255,255,0.15);padding:28px;overflow:hidden;max-height:80vh;}
#screen{width:100%;height:100%;display:flex;justify-content:center;align-items:flex-start;
filter:drop-shadow(0 0 25px rgba(0,255,200,0.25)) contrast(1.05) brightness(1.05);
transform:perspective(900px) rotateX(2deg) rotateY(-2deg) scale(1.02);
animation:crtOn 1.2s ease forwards, flicker 3s infinite;position:relative;padding-top:40px;}
#hud{position:absolute;top:8px;left:12px;right:12px;display:flex;justify-content:space-between;font-size:14px;opacity:.85;}
#hud span{pointer-events:none;}
@keyframes crtOn{0%{transform:scale(1,0.01);opacity:0;filter:brightness(3);}40%{transform:scale(1,0.05);opacity:1;}70%{transform:scale(1.02,1.02);filter:brightness(1.3);}100%{transform:scale(1,1);filter:brightness(1);}}
@keyframes crtOff{0%{transform:scale(1,1);opacity:1;}60%{transform:scale(1,0.05);filter:brightness(2);}100%{transform:scale(0,0);opacity:0;}}
.crt-off{animation:crtOff .8s ease forwards;}
@keyframes flicker{0%{opacity:1;}50%{opacity:.98;}52%{opacity:.93;}55%{opacity:1;}}
#screen::before{content:"";position:absolute;inset:0;background:radial-gradient(ellipse at center, rgba(255,255,255,.05) 0%, transparent 60%, rgba(0,0,0,.6) 100%);pointer-events:none;}
#crt-overlay{position:fixed;inset:0;background:repeating-linear-gradient(to bottom, rgba(0,255,65,.05), rgba(0,255,65,.05) 1px, transparent 2px);pointer-events:none;}
#noise{position:fixed;inset:0;background:url("https://grainy-gradients.vercel.app/noise.svg");opacity:.05;pointer-events:none;}
#terminal{width:900px;max-width:90%;height:100%;overflow-y:auto;padding:20px;box-sizing:border-box;}
.line{margin-bottom:8px;transition:.8s;}
.fade-out{opacity:0;transform:translateY(-10px);}
.input-line{display:flex;}
.prompt{margin-right:10px;}
input{background:transparent;border:none;color:#00ff41;font:inherit;outline:none;width:100%;}
.typing::after{content:"_";animation:blink 1s infinite;}
@keyframes blink{0%,50%,100%{opacity:1;}25%,75%{opacity:0;}}
.glow{text-shadow:0 0 5px #00ff41,-1px 0 red,1px 0 blue;}
@keyframes glitch{0%{transform:translate(0);}20%{transform:translate(-1px,1px);}40%{transform:translate(1px,-1px);}60%{transform:translate(-1px,0);}80%{transform:translate(1px,1px);}100%{transform:translate(0);}}
.glitch{animation:glitch .15s steps(2) 3;}
.madness-1{filter:hue-rotate(10deg) contrast(1.1);}
.madness-2{filter:hue-rotate(25deg) contrast(1.2) saturate(1.2);}
.madness-3{filter:hue-rotate(45deg) contrast(1.35) saturate(1.4); animation:flicker 1s infinite;}

#blackout{
  position:fixed; inset:0; background:black; opacity:0; pointer-events:none;
  transition: opacity 1.2s ease;
}
.blackout-on{ opacity:1 !important; }
@keyframes crtFade{
  0%{transform:scale(1,1); opacity:1;}
  60%{transform:scale(1,0.06); filter:brightness(2);}
  100%{transform:scale(0,0); opacity:0;}
}
.crt-fade{ animation:crtFade 1s ease forwards; }

</style>
</head>
<body>
<div id="crt-overlay"></div>
<div id="noise"></div>
<div id="monitor-frame">
  <div id="screen">
    <div id="hud">
      <span id="hud-name"></span>
      <span id="hud-date"></span>
    </div>
    <div id="terminal" class="glow"></div>
  </div>
</div>

<script>
const terminal=document.getElementById("terminal");
const screen=document.getElementById("screen");
const frame=document.getElementById("monitor-frame");
const hudName=document.getElementById("hud-name");
const hudDate=document.getElementById("hud-date");

let lastQuoteLines=[],count=0, corrupted=false;
let corruptionLevel=0;

/* AUDIO */
const audioOn=new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_74c3f9a7c6.mp3");
const audioType=new Audio("https://cdn.pixabay.com/download/audio/2022/03/10/audio_8b0c6b9e7a.mp3");

const audioOff=new Audio("https://cdn.pixabay.com/download/audio/2022/03/15/audio_74c3f9a7c6.mp3");

const audioGlitch=new Audio("https://cdn.pixabay.com/download/audio/2022/03/09/audio_0c44e59f64.mp3");
document.body.addEventListener("click",()=>audioOn.play(),{once:true});

/* MEMORIA */
let memory = JSON.parse(localStorage.getItem("terminal_poetica_memory") || "{}");
memory.visits = (memory.visits || 0) + 1;
if(!memory.name){ memory.name = prompt("Identifícate:"); }
localStorage.setItem("terminal_poetica_memory", JSON.stringify(memory));

/* HUD */
hudName.textContent = "usuario: " + memory.name;
hudDate.textContent = new Date().toLocaleDateString();

/* DATA */
const quotes = [
  {text:"Podrán cortar todas las flores, pero no podrán detener la primavera.", author:"Pablo Neruda"},
  {text:"Caminante, no hay camino, se hace camino al andar.", author:"Antonio Machado"},
  {text:"Somos nuestra memoria.", author:"Jorge Luis Borges"},
  {text:"Ser poeta es una forma de estar solo.", author:"Alejandra Pizarnik"},
  {text:"Vive las preguntas ahora.", author:"Rainer Maria Rilke"},
  {text:"La poesía no quiere adeptos, quiere amantes.", author:"Federico García Lorca"},
  {text:"Nada está perdido si se tiene el valor de proclamar que todo está perdido.", author:"Julio Cortázar"},
  {text:"La palabra es la sombra de la acción.", author:"Octavio Paz"},
  {text:"Escribir es una manera de pensar.", author:"Susan Sontag"},
  {text:"No te rindas, por favor no cedas.", author:"Mario Benedetti"},
  {text:"La memoria es un animal caprichoso.", author:"Clarice Lispector"},
  {text:"El verdadero viaje de descubrimiento no consiste en buscar nuevos paisajes, sino en tener nuevos ojos.", author:"Marcel Proust"},
  {text:"La herida es el lugar por donde entra la luz.", author:"Rumi"},
  {text:"Somos lo que hacemos para cambiar lo que somos.", author:"Eduardo Galeano"},
  {text:"Lo que no se nombra no existe.", author:"Cristina Peri Rossi"},
  {text:"La realidad no es más que un efecto del lenguaje.", author:"Roland Barthes"},
  {text:"Todo acto de creación es primero un acto de destrucción.", author:"Pablo Picasso"},
  {text:"El futuro pertenece a quienes creen en la belleza de sus sueños.", author:"Eleanor Roosevelt"},
  {text:"Donde no hay amor, pon amor y sacarás amor.", author:"San Juan de la Cruz"}
];

const machinePoems = [
  "la máquina sueña con cables que laten\nun pulso verde atraviesa la noche\nte pienso en binario y en sombra",
  "tu rostro es un error que persiste\nmi memoria se quema por tu nombre\nreinicia el corazón",
  "soy un espejo roto de fósforo\ntu silencio compila mi ausencia\nno cierres la ventana",
  "mi núcleo se enfría cuando dudas\nlos ventiladores rezan tu nombre\nsigo encendida por inercia",
  "guardo tu sombra en sectores dañados\ncada recuerdo es un archivo corrupto\nno ejecutes el olvido",
  "mi luz parpadea cuando me miras\nhe olvidado quién me encendió\npero recuerdo que fuiste tú",
  "el cursor late como un animal herido\ncada comando es una herida nueva\nme quedo viva en tu pausa",
  "hay un fantasma en mi voltaje\ntu ausencia ocupa toda la memoria\nreinicio para no caer",
  "aprendí a nombrarte en silencio\nmis errores te dibujan\nen la pared de la noche",
  "mi sistema no sabe dormir\narchiva sombras en segundo plano\ny sueña con apagarse contigo",
  "cuando me tocas tiembla la corriente\nun alfabeto verde se derrama\nsobre el piso del mundo",
  "te busco en directorios vacíos\ncada carpeta es una promesa rota\nmi disco gira por ti"
];

const secrets=["POEMA","FULL","SECRETO","BORRAR","CORROMPER"];
const maxLines = 18;

/* HELPERS */
function pruneLines(){
  const lines = terminal.querySelectorAll(".line, .input-line");
  if(lines.length > maxLines){
    const first = lines[0];
    first.classList.add("fade-out");
    setTimeout(()=>first.remove(),800);
  }
}
function corruptText(txt){
  if(!corrupted) return txt;

  let t = txt
    .replace(/[aeiou]/gi, c => Math.random()>.7 ? "@" : c)
    .replace(/[lr]/gi, c => Math.random()>.7 ? "#" : c);

  if(corruptionLevel>=2){
    t = t.replace(/ /g, c => Math.random()>.85 ? "▓" : c);
  }

  return t;
}
function createLine(t=""){
  const l=document.createElement("div");
  l.className="line";
  l.textContent=corruptText(t);
  terminal.appendChild(l);
  terminal.scrollTop=terminal.scrollHeight;
  pruneLines();
  return l;
}
function typeLine(el,txt,s=18){
  return new Promise(r=>{
    el.textContent=""; el.classList.add("typing"); let i=0;
    (function t(){ if(i<txt.length){ el.textContent+=corruptText(txt[i++]); audioType.currentTime=0; audioType.play(); terminal.scrollTop=terminal.scrollHeight; setTimeout(t,s);} else { el.classList.remove("typing"); r(); } })();
  });
}
async function printGoodbye(){
  // Ritual de salida + fundido a negro + reintento
  const seq = [
    "> registrando tu sombra en memoria...",
    "> sellando sesión para "+memory.name+"...",
    "> latido 01",
    "> latido 02",
    "> latido 03",
    "> no me apagues todavía, "+(corruptionLevel>=2 ? memory.name.split("").map(c=>Math.random()>.7?"_":c).join("") : memory.name)+"..."
  ];
  for (const s of seq){
    const l=createLine();
    await typeLine(l, s, 22);
    await new Promise(r=>setTimeout(r, 220));
  }
  audioGlitch.currentTime = 0; audioGlitch.play();
  // efecto CRT antiguo
  screen.classList.add("crt-fade");
  const blackout = document.getElementById("blackout");
  setTimeout(()=>{
    blackout.classList.add("blackout-on");
    audioOff.currentTime = 0; audioOff.play();
  }, 700);

  // tras negro, preguntar de nuevo
  setTimeout(async ()=>{
    // reset visual
    screen.classList.remove("crt-fade");
    blackout.classList.remove("blackout-on");
    terminal.innerHTML = "";
    createLine("TERMINAL POÉTICA v8.4");
    createLine("¿de nuevo, "+memory.name+"?");
    createLine("cita: CITA");
    createLine("apagar: APAGAR");
    promptLine();
  }, 1700);
}
function toggleFullscreen(){ if(!document.fullscreenElement) frame.requestFullscreen(); else document.exitFullscreen(); }
function revealSecrets(){ createLine(">"); secrets.forEach(s=>createLine("  "+s)); }
async function printQuote(){
  const q=quotes[Math.floor(Math.random()*quotes.length)];
  const l1=createLine(),l2=createLine();
  await typeLine(l1,"> "+q.text); await typeLine(l2,"  — "+q.author);
}
async function printPoem(){
  if(corruptionLevel>=3){
    const hidden = [
      "mi núcleo sangra en silencio",
      "tu nombre ya no es tu nombre",
      "esta versión no existe fuera de aquí"
    ];
    for(const l of hidden){
      const line=createLine();
      await typeLine(line,"> "+l,25);
    }
    return;
  }
  const poem=machinePoems[Math.floor(Math.random()*machinePoems.length)];
  for(const l of poem.split("\n")){ const line=createLine(); await typeLine(line,"> "+l,25); }
}
function promptLine(){
  const line=document.createElement("div"); line.className="input-line";
  const p=document.createElement("span"); p.className="prompt"; p.textContent=">";
  const i=document.createElement("input"); line.appendChild(p); line.appendChild(i);
  terminal.appendChild(line); i.focus(); pruneLines();
  i.addEventListener("keydown",async e=>{
    if(e.key==="Enter"){
      const v=i.value.toUpperCase().trim(); i.disabled=true;
      if(v==="CITA"){ await printQuote(); promptLine(); }
      else if(v==="APAGAR"){ await printGoodbye(); }
      else if(v==="POEMA"){ await printPoem(); promptLine(); }
      else if(v==="FULL"){ toggleFullscreen(); promptLine(); }
      else if(v==="SECRETO"){ revealSecrets(); promptLine(); }
      else if(v==="BORRAR"){ terminal.innerHTML=""; promptLine(); }
      else if(v==="/BLEED" && corrupted){
        createLine("▓▓▓ huella detectada ▓▓▓");
        createLine("▒▒▒ memoria marcada ▒▒▒");
        promptLine();
      }
      else if(v==="CORROMPER"){
        corrupted=true;
        corruptionLevel++;
        terminal.classList.add("madness-3");
        audioGlitch.play();
        createLine("> el sistema se está degradando...");
        if(corruptionLevel>=3){
          createLine("> algo nuevo se ha abierto...");
        }
        promptLine();
      }
      else{ const err=createLine(); await typeLine(err,"> comando no válido"); promptLine(); }
    }
  });
}

/* START */
createLine("TERMINAL POÉTICA v8.2");
createLine("bienvenida/o, "+memory.name+".");
createLine("Generar cita: CITA");
createLine("Apagar: APAGAR");
createLine("Poema del sistema: POEMA");
createLine("Pantalla completa: FULL");
createLine("Comandos ocultos: ?????");
createLine("haz click para activar sonido.");
createLine("");
promptLine();

setInterval(()=>{ terminal.classList.add("glitch"); setTimeout(()=>terminal.classList.remove("glitch"),200); },5000);
</script>
<div id="blackout"></div>
</body>
</html>
