<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Quero Polpa - Pedido Online</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Poppins', sans-serif; margin: 0; padding: 0; background: #fff5f5; color: #333; }
    header, footer { background: #e60023; color: #fff; text-align: center; padding: 20px; }
    .container { max-width: 600px; margin: 20px auto; padding: 20px; background: #fff; border-radius: 10px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); animation: fadeIn 1s ease; }
    h1, h2 { margin: 10px 0; }
    label { margin-top: 10px; display: block; }
    input, select { width: 100%; padding: 10px; margin-top: 5px; border: 1px solid #ccc; border-radius: 5px; }
    button { background: #e60023; color: #fff; border: none; padding: 12px 20px; border-radius: 5px; cursor: pointer; transition: transform 0.3s ease; }
    button:hover { transform: scale(1.05); }
    .pix-section, .frete-info { margin-top: 20px; }
    .qr-code { text-align: center; margin: 10px 0; }
    @keyframes fadeIn { from {opacity: 0; transform: translateY(-20px);} to {opacity: 1; transform: translateY(0);} }
  </style>
</head>
<body>

<header>
  <h1>Quero Polpa</h1>
  <p>Faça seu pedido online!</p>
</header>

<div class="container">
  <form id="pedidoForm">
    <label>Nome:</label>
    <input type="text" id="nome" required>

    <label>Endereço:</label>
    <input type="text" id="endereco" required>

    <label>Produto:</label>
    <select id="produto">
      <option>Polpa de Açaí</option>
      <option>Polpa de Cupuaçu</option>
      <option>Polpa de Graviola</option>
    </select>

    <label>Quantidade (kg):</label>
    <input type="number" id="quantidade" required>

    <button type="button" onclick="obterLocalizacaoCliente()">Calcular Frete</button>

    <div class="frete-info" id="freteInfo"></div>

    <div class="pix-section">
      <h2>Pagamento via Pix</h2>
      <p>Chave Pix: <b id="pixChave">rodrigo.mascena02@gmail.com</b></p>
      <button type="button" onclick="copiarPix()">Copiar Chave Pix</button>
      <div class="qr-code">
        <img id="qrCodePix" src="" alt="QR Code Pix" width="200">
      </div>
    </div>

    <button type="button" onclick="enviarPedido()">Enviar Pedido via WhatsApp</button>
  </form>
</div>

<footer>
  Travessa das Jaqueiras, nº 5, Paripe, Salvador - BA | Taxa: R$ 2,50/km
</footer>

<script>
  const apiKey = 'SUA_API_KEY_AQUI';
  const precoPorKm = 2.50;
  const estabelecimento = [-38.4931, -12.8926];
  const pixChave = 'rodrigo.mascena02@gmail.com';
  let dadosEntrega = { distanciaKm: 0, duracaoMin: 0, taxa: 0 };

  document.getElementById('qrCodePix').src = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(pixChave)}`;

  function copiarPix() {
    navigator.clipboard.writeText(pixChave).then(() => {
      alert('Chave Pix copiada com sucesso!');
    });
  }

  function obterLocalizacaoCliente() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(position => {
        const cliente = [position.coords.longitude, position.coords.latitude];
        calcularDistanciaETempo(cliente, estabelecimento);
      }, () => {
        alert('Não foi possível obter sua localização.');
      });
    } else {
      alert('Geolocalização não suportada.');
    }
  }

  function calcularDistanciaETempo(origem, destino) {
    const url = 'https://api.openrouteservice.org/v2/directions/driving-car/geojson';
    const body = { coordinates: [origem, destino] };

    fetch(url, {
      method: 'POST',
      headers: { 'Authorization': apiKey, 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    })
    .then(resp => resp.json())
    .then(data => {
      const sum = data.features[0].properties.summary;
      const distanciaKm = (sum.distance / 1000).toFixed(2);
      const duracaoMin = (sum.duration / 60).toFixed(1);
      const taxaEntrega = (distanciaKm * precoPorKm).toFixed(2);

      dadosEntrega = { distanciaKm, duracaoMin, taxa: taxaEntrega };
      document.getElementById('freteInfo').innerHTML = 
        `Distância: ${distanciaKm} km<br>Tempo: ${duracaoMin} min<br>Taxa: R$ ${taxaEntrega}`;
    })
    .catch(() => alert('Erro ao calcular a distância.'));
  }

  function enviarPedido() {
    const nome = document.getElementById('nome').value;
    const endereco = document.getElementById('endereco').value;
    const produto = document.getElementById('produto').value;
    const quantidade = parseFloat(document.getElementById('quantidade').value);

    if (quantidade >= 30) {
      var tipo = "Atacado";
    } else {
      var tipo = "Varejo";
    }

    const mensagem = `Olá, meu nome é ${nome}. Quero pedir:\n` +
                     `Produto: ${produto}\n` +
                     `Quantidade: ${quantidade} kg (${tipo})\n` +
                     `Endereço: ${endereco}\n` +
                     `Distância: ${dadosEntrega.distanciaKm} km\n` +
                     `Tempo estimado: ${dadosEntrega.duracaoMin} min\n` +
                     `Taxa de entrega: R$ ${dadosEntrega.taxa}\n` +
                     `Pagamento via Pix: ${pixChave}`;

    const numeroWhatsApp = '5571993571115';
    window.open(`https://wa.me/${numeroWhatsApp}?text=${encodeURIComponent(mensagem)}`, '_blank');
  }
</script>

</body>
</html>
