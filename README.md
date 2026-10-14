# gestao-insumos<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gestão de Compras & Insumos - Pelegrino</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 text-gray-800 font-sans p-4 md:p-6 max-w-5xl mx-auto">

  <!-- CABEÇALHO -->
  <header class="mb-6 flex justify-between items-center bg-slate-900 text-white p-4 rounded-xl shadow-md">
    <div>
      <h1 class="text-xl font-bold">Pelegrino - Gestão de Insumos</h1>
      <p class="text-xs text-slate-400">Controle de Notas, Preços e Teto dos 30%</p>
    </div>
    <span class="bg-emerald-500 text-xs font-semibold px-2.5 py-1 rounded text-white">App 2.0 (Isolado)</span>
  </header>

  <!-- PAINEL DOS 30% DE INSUMOS -->
  <section class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
    <div class="bg-white p-4 rounded-xl shadow border border-slate-200">
      <label class="block text-xs font-bold text-slate-500 uppercase mb-1">Faturamento Bruto (Mês)</label>
      <div class="flex items-center">
        <span class="text-gray-400 mr-1 font-bold">R$</span>
        <input type="number" id="faturamentoInput" placeholder="0.00" onchange="calcularInsumos()" class="w-full text-xl font-bold text-slate-800 focus:outline-none">
      </div>
      <span class="text-[10px] text-gray-400">Digite e dê Enter/clique fora</span>
    </div>

    <div class="bg-white p-4 rounded-xl shadow border border-slate-200">
      <span class="block text-xs font-bold text-slate-500 uppercase mb-1">Teto Máximo (30%)</span>
      <div class="text-xl font-bold text-blue-600" id="tetoValor">R$ 0,00</div>
      <span class="text-[10px] text-gray-400">Limite seguro para compras</span>
    </div>

    <div class="bg-white p-4 rounded-xl shadow border border-slate-200" id="cardStatus">
      <span class="block text-xs font-bold text-slate-500 uppercase mb-1">Gastos com Notas no Mês</span>
      <div class="text-xl font-bold text-slate-800" id="gastosValor">R$ 0,00</div>
      <span class="text-xs font-bold mt-1 block" id="statusTexto">Saldo Restante: R$ 0,00</span>
    </div>
  </section>

  <!-- ABAS DE NAVEGAÇÃO -->
  <div class="flex border-b border-gray-300 mb-6 space-x-2">
    <button onclick="mudarAba('notas')" id="btnAbaNotas" class="py-2 px-4 font-semibold text-blue-600 border-b-2 border-blue-600 focus:outline-none">Lançar Notas</button>
    <button onclick="mudarAba('historico')" id="btnAbaHistorico" class="py-2 px-4 font-semibold text-gray-500 focus:outline-none">Comparador de Preços</button>
  </div>

  <!-- ABA 1: LANÇAMENTO DE NOTAS / ITENS -->
  <main id="abaNotas" class="space-y-6">
    <div class="bg-white p-5 rounded-xl shadow border border-slate-200">
      <h2 class="text-lg font-bold mb-4 text-slate-800">Cadastrar Nova Entrada de Produto</h2>
      <form id="formItem" onsubmit="salvarItem(event)" class="grid grid-cols-1 md:grid-cols-2 gap-4">
        
        <div>
          <label class="block text-xs font-semibold mb-1">Nome do Insumo</label>
          <input type="text" id="nomeItem" placeholder="Ex: Arroz, Óleo, Laranja" required class="w-full p-2 border rounded text-sm focus:ring-2 focus:ring-blue-500 outline-none">
        </div>

        <div>
          <label class="block text-xs font-semibold mb-1">Tipo de Compra</label>
          <select id="tipoCompra" onchange="ajustarCamposTipo()" class="w-full p-2 border rounded text-sm bg-white focus:ring-2 focus:ring-blue-500 outline-none">
            <option value="fardo">Fardo / Caixa (com conversão)</option>
            <option value="unidade">Unidade Avulsa (Mercado/Feira)</option>
            <option value="semanal">Saco / Embalagem Semanal (Hortifrúti)</option>
          </select>
        </div>

        <div>
          <label class="block text-xs font-semibold mb-1">Valor Total Pago na Nota (R$)</label>
          <input type="number" step="0.01" id="valorTotal" placeholder="100.00" required class="w-full p-2 border rounded text-sm focus:ring-2 focus:ring-blue-500 outline-none">
        </div>

        <div id="campoQtdConteudo">
          <label class="block text-xs font-semibold mb-1">Quantas Unidades Vêm Dentro?</label>
          <input type="number" id="qtdConteudo" placeholder="Ex: 10" value="1" class="w-full p-2 border rounded text-sm focus:ring-2 focus:ring-blue-500 outline-none">
        </div>

        <div>
          <label class="block text-xs font-semibold mb-1">Fornecedor / Loja</label>
          <input type="text" id="fornecedor" placeholder="Ex: Atacadão, Mercado X" class="w-full p-2 border rounded text-sm focus:ring-2 focus:ring-blue-500 outline-none">
        </div>

        <div>
          <label class="block text-xs font-semibold mb-1">Data da Nota/Compra</label>
          <input type="date" id="dataNota" required class="w-full p-2 border rounded text-sm focus:ring-2 focus:ring-blue-500 outline-none">
        </div>

        <div class="md:col-span-2">
          <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2.5 rounded-lg shadow transition">Salvar Item no Estoque</button>
        </div>
      </form>
    </div>

    <!-- TABELA DE ENTRADAS RECENTES -->
    <div class="bg-white p-5 rounded-xl shadow border border-slate-200">
      <h3 class="text-md font-bold mb-3 text-slate-800">Últimas Entradas Salvas</h3>
      <div class="overflow-x-auto">
        <table class="w-full text-left text-sm">
          <thead class="bg-slate-100 text-slate-600 uppercase text-[10px]">
            <tr>
              <th class="p-2">Data</th>
              <th class="p-2">Produto</th>
              <th class="p-2">Tipo</th>
              <th class="p-2">Valor Total</th>
              <th class="p-2">Preço Unitário</th>
              <th class="p-2">Ação</th>
            </tr>
          </thead>
          <tbody id="tabelaEntradas" class="divide-y divide-gray-200">
            <!-- Injetado por JS -->
          </tbody>
        </table>
      </div>
    </div>
  </main>

  <!-- ABA 2: HISTÓRICO DE COMPARATIVO DE PREÇOS -->
  <main id="abaHistorico" class="hidden space-y-6">
    <div class="bg-white p-5 rounded-xl shadow border border-slate-200">
      <h2 class="text-lg font-bold mb-4 text-slate-800">Comparar Histórico de Preços</h2>
      <div class="mb-4">
        <label class="block text-xs font-semibold mb-1">Pesquisar Insumo</label>
        <input type="text" id="buscaInsumo" placeholder="Digite o nome do produto (Ex: Arroz)" oninput="buscarHistorico()" class="w-full p-2 border rounded text-sm outline-none focus:ring-2 focus:ring-blue-500">
      </div>

      <div id="resultadoHistorico" class="space-y-3">
        <p class="text-xs text-gray-500 italic">Digite o nome de um insumo acima para verificar as altas ou baixas de preço.</p>
      </div>
    </div>
  </main>

  <!-- SCRIPT LÓGICA EM JAVASCRIPT -->
  <script>
    // Usa uma chave EXCLUSIVA para não misturar com o app antigo!
    const CHAVE_FATURAMENTO = 'PELEGRINO_FATURAMENTO_APP2';
    const CHAVE_ENTRADAS = 'PELEGRINO_ENTRADAS_APP2';

    document.getElementById('dataNota').valueAsDate = new Date();

    window.onload = function() {
      const fatSalvo = localStorage.getItem(CHAVE_FATURAMENTO);
      if (fatSalvo) {
        document.getElementById('faturamentoInput').value = fatSalvo;
      }
      calcularInsumos();
      renderizarTabela();
    };

    function ajustarCamposTipo() {
      const tipo = document.getElementById('tipoCompra').value;
      const campoQtd = document.getElementById('campoQtdConteudo');
      if (tipo === 'fardo') {
        campoQtd.classList.remove('hidden');
      } else {
        campoQtd.classList.add('hidden');
        document.getElementById('qtdConteudo').value = 1;
      }
    }

    function calcularInsumos() {
      const faturamento = parseFloat(document.getElementById('faturamentoInput').value) || 0;
      localStorage.setItem(CHAVE_FATURAMENTO, faturamento);

      const teto30 = faturamento * 0.30;
      document.getElementById('tetoValor').innerText = `R$ ${teto30.toFixed(2)}`;

      const entradas = JSON.parse(localStorage.getItem(CHAVE_ENTRADAS) || '[]');
      const totalGastos = entradas.reduce((acc, item) => acc + parseFloat(item.valorTotal), 0);
      document.getElementById('gastosValor').innerText = `R$ ${totalGastos.toFixed(2)}`;

      const saldo = teto30 - totalGastos;
      const statusTexto = document.getElementById('statusTexto');
      const cardStatus = document.getElementById('cardStatus');

      if (saldo >= 0) {
        statusTexto.innerText = `Saldo Restante: R$ ${saldo.toFixed(2)}`;
        statusTexto.className = "text-xs font-bold mt-1 block text-emerald-600";
        cardStatus.className = "bg-white p-4 rounded-xl shadow border-2 border-emerald-500";
      } else {
        statusTexto.innerText = `Estourou a Meta: -R$ ${Math.abs(saldo).toFixed(2)}`;
        statusTexto.className = "text-xs font-bold mt-1 block text-rose-600";
        cardStatus.className = "bg-white p-4 rounded-xl shadow border-2 border-rose-500";
      }
    }

    function salvarItem(e) {
      e.preventDefault();
      const nome = document.getElementById('nomeItem').value.trim().toLowerCase();
      const tipo = document.getElementById('tipoCompra').value;
      const valorTotal = parseFloat(document.getElementById('valorTotal').value);
      const qtdConteudo = parseInt(document.getElementById('qtdConteudo').value) || 1;
      const fornecedor = document.getElementById('fornecedor').value;
      const dataNota = document.getElementById('dataNota').value;

      const precoUnitario = valorTotal / qtdConteudo;

      const novoItem = {
        id: Date.now(),
        nome,
        tipo,
        valorTotal,
        qtdConteudo,
        precoUnitario,
        fornecedor,
        dataNota
      };

      const entradas = JSON.parse(localStorage.getItem(CHAVE_ENTRADAS) || '[]');
      entradas.unshift(novoItem);
      localStorage.setItem(CHAVE_ENTRADAS, JSON.stringify(entradas));

      document.getElementById('formItem').reset();
      document.getElementById('dataNota').valueAsDate = new Date();
      ajustarCamposTipo();

      calcularInsumos();
      renderizarTabela();
    }

    function excluirItem(id) {
      let entradas = JSON.parse(localStorage.getItem(CHAVE_ENTRADAS) || '[]');
      entradas = entradas.filter(i => i.id !== id);
      localStorage.setItem(CHAVE_ENTRADAS, JSON.stringify(entradas));
      calcularInsumos();
      renderizarTabela();
    }

    function renderizarTabela() {
      const entradas = JSON.parse(localStorage.getItem(CHAVE_ENTRADAS) || '[]');
      const tbody = document.getElementById('tabelaEntradas');
      tbody.innerHTML = '';

      if (entradas.length === 0) {
        tbody.innerHTML = `<tr><td colspan="6" class="p-4 text-center text-gray-400">Nenhum item cadastrado ainda.</td></tr>`;
        return;
      }

      entradas.forEach(item => {
        const tr = document.createElement('tr');
        tr.className = "hover:bg-slate-50";
        tr.innerHTML = `
          <td class="p-2">${item.dataNota}</td>
          <td class="p-2 font-bold capitalize">${item.nome}</td>
          <td class="p-2 text-xs text-gray-500">${item.tipo}</td>
          <td class="p-2 font-semibold">R$ ${item.valorTotal.toFixed(2)}</td>
          <td class="p-2 text-blue-600 font-bold">R$ ${item.precoUnitario.toFixed(2)} /un</td>
          <td class="p-2"><button onclick="excluirItem(${item.id})" class="text-xs text-rose-500 hover:underline">Remover</button></td>
        `;
        tbody.appendChild(tr);
      });
    }

    function mudarAba(aba) {
      if (aba === 'notas') {
        document.getElementById('abaNotas').classList.remove('hidden');
        document.getElementById('abaHistorico').classList.add('hidden');
        document.getElementById('btnAbaNotas').className = "py-2 px-4 font-semibold text-blue-600 border-b-2 border-blue-600 focus:outline-none";
        document.getElementById('btnAbaHistorico').className = "py-2 px-4 font-semibold text-gray-500 focus:outline-none";
      } else {
        document.getElementById('abaNotas').classList.add('hidden');
        document.getElementById('abaHistorico').classList.remove('hidden');
        document.getElementById('btnAbaNotas').className = "py-2 px-4 font-semibold text-gray-500 focus:outline-none";
        document.getElementById('btnAbaHistorico').className = "py-2 px-4 font-semibold text-blue-600 border-b-2 border-blue-600 focus:outline-none";
      }
    }

    function buscarHistorico() {
      const termo = document.getElementById('buscaInsumo').value.trim().toLowerCase();
      const divResultado = document.getElementById('resultadoHistorico');
      const entradas = JSON.parse(localStorage.getItem(CHAVE_ENTRADAS) || '[]');

      if (!termo) {
        divResultado.innerHTML = `<p class="text-xs text-gray-500 italic">Digite o nome de um insumo acima para verificar as altas ou baixas de preço.</p>`;
        return;
      }

      const filtrados = entradas.filter(i => i.nome.includes(termo)).sort((a,b) => new Date(a.dataNota) - new Date(b.dataNota));

      if (filtrados.length === 0) {
        divResultado.innerHTML = `<p class="text-xs text-rose-500">Nenhum histórico encontrado para "${termo}".</p>`;
        return;
      }

      let html = `<div class="space-y-2">`;
      for (let i = 0; i < filtrados.length; i++) {
        const item = filtrados[i];
        let indicacao = '';

        if (i > 0) {
          const anterior = filtrados[i-1].precoUnitario;
          const atual = item.precoUnitario;
          if (atual > anterior) {
            const pct = (((atual - anterior) / anterior) * 100).toFixed(1);
            indicacao = `<span class="text-xs font-bold text-rose-600 bg-rose-50 px-2 py-0.5 rounded">🔴 Subiu ${pct}% em relação à compra anterior</span>`;
          } else if (atual < anterior) {
            const pct = (((anterior - atual) / anterior) * 100).toFixed(1);
            indicacao = `<span class="text-xs font-bold text-emerald-600 bg-emerald-50 px-2 py-0.5 rounded">🟢 Baixou ${pct}% em relação à compra anterior</span>`;
          } else {
            indicacao = `<span class="text-xs text-gray-500 bg-gray-100 px-2 py-0.5 rounded">Mantendo o mesmo preço</span>`;
          }
        }

        html += `
          <div class="p-3 bg-slate-50 border rounded-lg flex justify-between items-center">
            <div>
              <span class="text-xs font-bold text-slate-500 block">${item.dataNota} - ${item.fornecedor || 'Fornecedor não informado'}</span>
              <span class="text-sm font-bold text-slate-800 capitalize">${item.nome}</span>
              <div class="mt-1">${indicacao}</div>
            </div>
            <div class="text-right">
              <span class="text-sm font-bold text-blue-600 block">R$ ${item.precoUnitario.toFixed(2)} /un</span>
              <span class="text-[10px] text-gray-400">Total Nota: R$ ${item.valorTotal.toFixed(2)}</span>
            </div>
          </div>
        `;
      }
      html += `</div>`;
      divResultado.innerHTML = html;
    }
  </script>
</body>
</html>
