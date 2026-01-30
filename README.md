[index.html](https://github.com/user-attachments/files/24951861/index.html)
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Sistema Online de Vendas</title>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
body { font-family: Arial; margin:0; background:#0d1117; color:#fff; }
header { background:#111827; padding:15px; text-align:center; font-size:22px; font-weight:bold; }
.section { display:none; padding:20px; max-width:1000px; margin:auto; }
input, select { width:100%; padding:8px; margin:5px 0; border-radius:6px; border:none; }
button { padding:10px; border:none; border-radius:6px; font-weight:bold; cursor:pointer; }
.grid { display:grid; grid-template-columns: repeat(auto-fit,minmax(180px,1fr)); gap:10px; }
.pedido { background:#1f2937; padding:10px; margin:10px 0; border-radius:8px; }
.total { color:#4ade80; font-weight:bold; }
</style>
</head>
<body>

<header>📦 Sistema Online de Vendas</header>

<div id="login" class="section" style="display:block;">
<h2>Login</h2>
<input id="user" placeholder="Usuário">
<input id="pass" type="password" placeholder="Senha">
<button onclick="login()">Entrar</button>
<button onclick="mostrarCadastro()">Criar Conta</button>
<p id="msg"></p>
</div>

<div id="cadastro" class="section">
<h2>Cadastro</h2>
<input id="newUser" placeholder="Usuário">
<input id="newPass" type="password" placeholder="Senha">
<button onclick="cadastrar()">Cadastrar</button>
<button onclick="voltarLogin()">Voltar</button>
<p id="msg2"></p>
</div>

<div id="sistema" class="section">
<button onclick="logout()">Sair</button>
<h2>Nova Venda</h2>
<select id="tipo">
<option value="normal">Cliente Normal</option>
<option value="parceiro">Parceiro</option>
</select>

<div class="grid">
<input type="number" id="fuzil" placeholder="Fuzil">
<input type="number" id="sniper" placeholder="Sniper">
<input type="number" id="smg" placeholder="SMG">
<input type="number" id="pistola" placeholder="Pistola">
<input type="number" id="dinamite" placeholder="Dinamite">
<input type="number" id="c4" placeholder="C4">
</div>

<button onclick="registrarPedido()">Registrar Pedido</button>
<p id="resumo"></p>

<h2 id="painelTitulo" style="display:none;">📋 Painel do Operador</h2>
<div id="pedidos"></div>
</div>

<script>
const supabase = window.supabase.createClient(
  "https://gznoajgxxklgspeobgqj.supabase.co",
  "sb_publishable_WDaRdocyOlxyOPtb3LkWsA_l7_U0Nnc"
);

let usuarioAtual = null;
let tipoConta = null;

const precos = {
normal:{fuzil:500000,sniper:200000,smg:130000,pistola:80000,dinamite:10000,c4:25000},
parceiro:{fuzil:400000,sniper:160000,smg:105000,pistola:64000,dinamite:6000,c4:20000}
};

function mostrarCadastro(){ login.style.display="none"; cadastro.style.display="block"; }
function voltarLogin(){ cadastro.style.display="none"; login.style.display="block"; }

async function cadastrar(){
let { error } = await supabase.from("usuarios").insert([
{ usuario:newUser.value, senha:newPass.value, tipo:"cliente" }
]);
msg2.innerText = error ? error.message : "Conta criada!";
}

async function login(){
let { data } = await supabase.from("usuarios")
.select("*").eq("usuario", user.value).eq("senha", pass.value).single();

if(!data){ msg.innerText="Login inválido"; return; }

usuarioAtual = data.usuario;
tipoConta = data.tipo;

login.style.display="none";
sistema.style.display="block";

if(tipoConta==="operador"){ painelTitulo.style.display="block"; carregarPedidos(); }
}

function logout(){ location.reload(); }

async function registrarPedido(){
let tipo=document.getElementById("tipo").value;
let total=0;
let itens=[];

for(let item in precos[tipo]){
let qtd=Number(document.getElementById(item).value)||0;
if(qtd>0){
let valor=qtd*precos[tipo][item];
total+=valor;
itens.push(item+" x"+qtd);
}
}

await supabase.from("pedidos").insert([
{ cliente:usuarioAtual, itens:itens.join(", "), total:total, data:new Date().toLocaleString() }
]);

resumo.innerHTML="Pedido registrado! Total: R$ "+total.toLocaleString();
if(tipoConta==="operador") carregarPedidos();
}

async function carregarPedidos(){
let { data } = await supabase.from("pedidos").select("*").order("id",{ascending:false});
pedidos.innerHTML="";
data.forEach(p=>{
pedidos.innerHTML+=`<div class="pedido">👤 ${p.cliente}<br>📦 ${p.itens}<br><span class="total">R$ ${p.total}</span></div>`;
});
}
</script>

</body>
</html>
