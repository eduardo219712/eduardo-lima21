# eduardo-lima21
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Organizador Pessoal</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
        }
        .container {
            width: 80%;
            margin: auto;
        }
        .login-form, .dashboard {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            margin-top: 50px;
        }
        .dashboard {
            display: none;
        }
        input[type="password"], input[type="text"], input[type="number"], button, select {
            padding: 10px;
            margin: 10px 0;
            width: 100%;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        .section button {
            margin-top: 10px;
        }
        .edit-btn, .remove-btn {
            cursor: pointer;
            color: blue;
            margin-left: 10px;
        }
        .edit-btn:hover, .remove-btn:hover {
            text-decoration: underline;
        }
        ul {
            padding-left: 20px;
        }
        li {
            margin-bottom: 10px;
        }
        .error {
            color: red;
            font-size: 0.9em;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="login-form">
        <h2>Entrar no Sistema</h2>
        <input type="password" id="password" placeholder="Digite sua senha">
        <button onclick="login()">Entrar</button>
        <p id="error-message" class="error" style="display: none;">Senha incorreta!</p>
    </div>

    <div class="dashboard">
        <h2>Painel de Controle</h2>
        <button onclick="showSection('financas')">Finanças</button>
        <button onclick="showSection('pessoal')">Vida Pessoal</button>
        <button onclick="showSection('grafico')">Gráficos</button>
        <button onclick="exportData()">Exportar Dados</button>

        <div id="financas" class="section">
            <h3>Finanças</h3>
            <input type="text" id="descricao" placeholder="Descrição">
            <input type="number" id="valor" placeholder="Valor">
            <select id="categoria">
                <option value="">Selecione uma categoria</option>
                <option value="Alimentação">Alimentação</option>
                <option value="Transporte">Transporte</option>
                <option value="Lazer">Lazer</option>
                <option value="Saúde">Saúde</option>
                <option value="Moradia">Moradia</option>
            </select>
            <button onclick="addFinanca()">Adicionar Finança</button>
            <ul id="lista-financas"></ul>
        </div>

        <div id="pessoal" class="section" style="display: none;">
            <h3>Vida Pessoal</h3>
            <input type="text" id="atividade" placeholder="Atividade Pessoal">
            <button onclick="addPessoal()">Adicionar Atividade</button>
            <ul id="lista-pessoal"></ul>
        </div>

        <div id="grafico" class="section" style="display: none;">
            <h3>Gráficos</h3>
            <canvas id="grafico-gastos" width="400" height="200"></canvas>
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
    const senhaCorreta = "219712";
    let financas = JSON.parse(localStorage.getItem('financas')) || [];
    let atividadesPessoais = JSON.parse(localStorage.getItem('atividadesPessoais')) || [];

    function login() {
        const senha = document.getElementById("password").value;
        if (senha === senhaCorreta) {
            document.querySelector(".login-form").style.display = "none";
            document.querySelector(".dashboard").style.display = "block";
            carregarFinancas();
            carregarAtividadesPessoais();
        } else {
            document.getElementById("error-message").style.display = "block";
        }
    }

    function showSection(section) {
        document.querySelectorAll('.section').forEach(sec => {
            sec.style.display = 'none';
        });
        document.getElementById(section).style.display = 'block';
    }

    function addFinanca() {
        const descricao = document.getElementById("descricao").value;
        const valor = parseFloat(document.getElementById("valor").value);
        const categoria = document.getElementById("categoria").value;
        
        if (descricao && !isNaN(valor) && categoria) {
            financas.push({ descricao, valor, categoria });
            localStorage.setItem('financas', JSON.stringify(financas));
            carregarFinancas();
        } else {
            alert("Por favor, preencha todos os campos corretamente.");
        }
    }

    function carregarFinancas() {
        const lista = document.getElementById("lista-financas");
        lista.innerHTML = "";
        financas.forEach((financa, index) => {
            const li = document.createElement("li");
            li.innerHTML = `${financa.descricao}: R$${financa.valor.toFixed(2)} - Categoria: ${financa.categoria} 
                <span class="edit-btn" onclick="editFinanca(${index})">Editar</span>
                <span class="remove-btn" onclick="removeFinanca(${index})">Remover</span>`;
            lista.appendChild(li);
        });
        gerarGrafico();
    }

    function editFinanca(index) {
        const descricao = prompt("Editar descrição:", financas[index].descricao);
        const valor = parseFloat(prompt("Editar valor:", financas[index].valor));
        const categoria = prompt("Editar categoria:", financas[index].categoria);
        
        if (descricao && !isNaN(valor) && categoria) {
            financas[index] = { descricao, valor, categoria };
            localStorage.setItem('financas', JSON.stringify(financas));
            carregarFinancas();
        }
    }

    function removeFinanca(index) {
        if (confirm("Deseja remover este item?")) {
            financas.splice(index, 1);
            localStorage.setItem('financas', JSON.stringify(financas));
            carregarFinancas();
        }
    }

    function addPessoal() {
        const atividade = document.getElementById("atividade").value;
        if (atividade) {
            atividadesPessoais.push(atividade);
            localStorage.setItem('atividadesPessoais', JSON.stringify(atividadesPessoais));
            carregarAtividadesPessoais();
        }
    }

    function carregarAtividadesPessoais() {
        const lista = document.getElementById("lista-pessoal");
        lista.innerHTML = "";
        atividadesPessoais.forEach((atividade, index) => {
            const li = document.createElement("li");
            li.innerHTML = `${atividade} 
                <span class="edit-btn" onclick="editPessoal(${index})">Editar</span>
                <span class="remove-btn" onclick="removePessoal(${index})">Remover</span>`;
            lista.appendChild(li);
        });
    }

    function editPessoal(index) {
        const atividade = prompt("Editar atividade:", atividadesPessoais[index]);
        if (atividade) {
            atividadesPessoais[index] = atividade;
            localStorage.setItem('atividadesPessoais', JSON.stringify(atividadesPessoais));
            carregarAtividadesPessoais();
        }
    }

    function removePessoal(index) {
        if (confirm("Deseja remover esta atividade?")) {
            atividadesPessoais.splice(index, 1);
            localStorage.setItem('atividadesPessoais', JSON.stringify(atividadesPessoais));
            carregarAtividadesPessoais();
        }
    }

    function gerarGrafico() {
        const ctx = document.getElementById("grafico-gastos").getContext("2d");
        
        const categorias = [...new Set(financas.map(f => f.categoria))];
        const valoresPorCategoria = categorias.map(categoria => {
            return financas.filter(f => f.categoria === categoria).reduce((sum, f) => sum + f.valor, 0);
        });
        
        new Chart(ctx, {
            type: 'pie',
            data: {
                labels: categorias,
                datasets: [{
                    label: 'Gastos por Categoria',
                    data: valoresPorCategoria,
                    backgroundColor: ['#ff5733', '#33ff57', '#3357ff', '#f0e130', '#c0c0c0'],
                }]
            }
        });
    }

    function exportData() {
        const financasCSV = financas.map(f => `${f.descricao},${f.valor.toFixed(2)},${f.categoria}`).join("\n");
        const atividadesCSV = atividadesPessoais.join("\n");

        const csvContent = "Finanças:\nDescrição,Valor,Categoria\n" + financasCSV + "\n\nVida Pessoal:\nAtividade\n" + atividadesCSV;

        const blob = new Blob([csvContent], { type: 'text/csv' });
        const link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = "dados_pessoais.csv";
        link.click();
    }
</script>

</body>
</html>
