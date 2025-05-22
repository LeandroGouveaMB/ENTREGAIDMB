<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Entrega de Identidades</title>
  <style>
    body { 
      font-family: Arial, sans-serif; 
      background-color: #ffffff;
      padding: 20px; 
      padding-top: 120px;
      margin: 0;
    }
    .form-section {
      display: flex; 
      align-items: flex-start; 
      gap: 10px;
      margin-bottom: 20px; 
      flex-wrap: wrap;
      position: fixed;
      top: 0; 
      left: 0; 
      width: 100%;
      background-color: #ffffff;
      padding: 20px; 
      box-sizing: border-box;
      z-index: 1000;
    }
    .form-section input[type="text"] {
      padding: 8px; 
      font-size: 16px; 
      width: 200px;
      background-color: #fff8e1; 
      border: 1px solid #ccc;
    }
    .form-section label {
      font-weight: bold; 
      font-size: 14px; 
      color: darkred;
    }
    .btn {
      padding: 10px; 
      font-size: 14px; 
      font-weight: bold;
      border: none; 
      cursor: pointer; 
      border-radius: 4px;
    }
    .btn-salvar { background-color: lightgreen; }
    .btn-assinar { background-color: #5fa8d3; color: white; }
    .btn-impossibilitado { background-color: #f4a261; }
    .btn-recibo { background-color: #f4f1a1; }
    .btn-autorizacao { background-color: #e0e0e0; }
    table {
      width: 100%; 
      border-collapse: collapse; 
      font-size: 14px;
    }
    th, td {
      border: 1px solid #888; 
      padding: 6px; 
      text-align: left;
    }
    th {
      background-color: #ddd; 
      font-weight: bold;
    }
  </style>
</head>
<body>

  <div class="form-section">
    <form name="entrega-identidades" data-netlify="true" id="formulario" method="POST" enctype="multipart/form-data">
      <input type="hidden" name="form-name" value="entrega-identidades">
      <label for="nip">NIP ou Registro do Titular</label>
      <input type="text" id="nip" name="nip" placeholder="Digite o NIP..." onkeydown="if(event.key === 'Enter') salvarRegistro()">
      <button type="button" class="btn btn-salvar" onclick="salvarRegistro()">Salvar Registro ✅</button>
      <button type="button" class="btn btn-assinar" onclick="abrirSeletorEPrograma()">Gravar Assinatura 🖊</button>
      <button type="button" class="btn btn-salvar" onclick="salvarEPular()">Salvar</button>
      <button type="button" class="btn btn-impossibilitado" onclick="marcarImpossibilitado()">Impossibilitado de Assinar</button>
      <button type="button" class="btn btn-recibo" onclick="gerarRecibo()">GERAR RECIBO</button>
      <button type="button" class="btn btn-autorizacao" onclick="marcarAutorizacao()">Trouxe Autorização</button>
      <!-- Campos ocultos para enviar data/hora, recebido por e autorização -->
      <input type="hidden" id="dataHora" name="dataHora">
      <input type="hidden" id="recebidoPor" name="recebidoPor">
      <input type="hidden" id="autorizacao" name="autorizacao">
    </form>
  </div>

  <table id="tabela">
    <thead>
      <tr>
        <th>NIP ou Registro do Titular</th>
        <th>Data/Hora da Entrega</th>
        <th>RECEBIDO POR:</th>
        <th>AUTORIZAÇÃO</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <input type="file" id="fileInput" name="imagem" accept="image/*" style="display: none" onchange="inserirImagem(this)">

  <script>
    let linhaAtual = null;

    // Manipulador de envio do formulário
    const form = document.getElementById("formulario");
    form.addEventListener("submit", async (event) => {
      event.preventDefault(); // Impede o comportamento padrão (abrir nova aba)
      const formData = new FormData(form);

      try {
        const response = await fetch("/", {
          method: "POST",
          body: formData
        });
        console.log("Dados enviados ao Netlify!");
        alert("Ação registrada com sucesso!");
      } catch (error) {
        console.error("Erro ao enviar dados:", error);
        alert("Erro ao registrar. Verifique o console.");
      }
    });

    async function salvarRegistro() {
      const nipInput = document.getElementById("nip");
      const nip = nipInput.value.trim();
      if (!nip) {
        alert("Digite um NIP!");
        return;
      }

      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
      const novaLinha = tabela.insertRow();
      const dataHora = new Date().toLocaleString("pt-BR");

      const nipCell = novaLinha.insertCell(0);
      nipCell.textContent = nip;
      nipCell.id = `nip-${nip}-${Date.now()}`;

      novaLinha.insertCell(1).textContent = dataHora;
      novaLinha.insertCell(2).innerHTML = "";
      novaLinha.insertCell(3).textContent = "";

      linhaAtual = novaLinha;

      // Preencher campos ocultos e disparar evento de submit
      document.getElementById("nip").value = nip;
      document.getElementById("dataHora").value = dataHora;
      document.getElementById("recebidoPor").value = "";
      document.getElementById("autorizacao").value = "";
      form.dispatchEvent(new Event("submit"));

      nipCell.scrollIntoView({ behavior: "smooth", block: "center" });
      nipInput.value = "";
      nipInput.focus();
    }

    function abrirSeletorEPrograma() {
      if (!linhaAtual) {
        alert("Salve o registro antes de capturar a assinatura.");
        return;
      }
      document.getElementById("fileInput").click();
    }

    async function inserirImagem(input) {
      const arquivo = input.files[0];
      if (!arquivo || !linhaAtual) return;

      const leitor = new FileReader();
      leitor.onload = async function(e) {
        const img = document.createElement("img");
        img.src = e.target.result;
        img.style.maxHeight = "50px";
        linhaAtual.cells[2].innerHTML = "";
        linhaAtual.cells[2].appendChild(img);

        // Preencher campos ocultos e disparar evento de submit
        document.getElementById("nip").value = linhaAtual.cells[0].textContent;
        document.getElementById("dataHora").value = linhaAtual.cells[1].textContent;
        document.getElementById("recebidoPor").value = "Imagem enviada";
        document.getElementById("autorizacao").value = linhaAtual.cells[3].textContent;
        const formData = new FormData(form);
        formData.append("imagem", arquivo);

        try {
          await fetch("/", {
            method: "POST",
            body: formData
          });
          console.log("Imagem enviada ao Netlify!");
          alert("Imagem registrada com sucesso!");
        } catch (error) {
          console.error("Erro ao enviar imagem:", error);
          alert("Erro ao registrar imagem. Verifique o console.");
        }
      };
      leitor.readAsDataURL(arquivo);
    }

    async function marcarImpossibilitado() {
      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
      if (tabela.rows.length === 0) {
        alert("Nenhum registro para marcar.");
        return;
      }

      const ultimaLinha = tabela.rows[tabela.rows.length - 1];
      ultimaLinha.cells[2].textContent = "Impossibilitado de Assinar";

      // Preencher campos ocultos e disparar evento de submit
      document.getElementById("nip").value = ultimaLinha.cells[0].textContent;
      document.getElementById("dataHora").value = ultimaLinha.cells[1].textContent;
      document.getElementById("recebidoPor").value = "Impossibilitado de Assinar";
      document.getElementById("autorizacao").value = ultimaLinha.cells[3].textContent;
      form.dispatchEvent(new Event("submit"));
    }

    async function marcarAutorizacao() {
      if (!linhaAtual) {
        alert("Salve o registro antes de marcar a autorização.");
        return;
      }

      linhaAtual.cells[3].textContent = "Trouxe Autorização";

      // Preencher campos ocultos e disparar evento de submit
      document.getElementById("nip").value = linhaAtual.cells[0].textContent;
      document.getElementById("dataHora").value = linhaAtual.cells[1].textContent;
      document.getElementById("recebidoPor").value = linhaAtual.cells[2].textContent || (linhaAtual.cells[2].innerHTML.includes("img") ? "Imagem enviada" : "");
      document.getElementById("autorizacao").value = "Trouxe Autorização";
      form.dispatchEvent(new Event("submit"));
    }

    function gerarRecibo() {
      const nip = prompt("Digite o NIP do registro que deseja gerar o recibo:");
      if (!nip) return;

      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
      for (let i = 0; i < tabela.rows.length; i++) {
        const linha = tabela.rows[i];
        if (linha.cells[0].textContent.trim() === nip.trim()) {
          const assinaturaCell = linha.cells[2];
          const assinatura = assinaturaCell.innerHTML || "Impossibilitado de Assinar";

          const hoje = new Date();
          const dia = String(hoje.getDate()).padStart(2, '0');
          const mes = String(hoje.getMonth() + 1).padStart(2, '0');
          const ano = hoje.getFullYear();
          const dataAtual = `${dia}/${mes}/${ano}`;

          const reciboHTML = `
            <html>
              <head>
                <title>Termo de Recebimento</title>
                <style>
                  body { 
                    font-family: sans-serif; 
                    padding: 10px; 
                    line-height: 1.4; 
                    width: 400px; 
                    margin: 0 auto; 
                    font-size: 12px; 
                  }
                  h2, h3, h4 { text-align: center; margin: 3px 0; }
                  h2 { font-size: 14px; }
                  h3 { font-size: 12px; }
                  h4 { font-size: 10px; }
                  h4 span { font-weight: bold; }
                  .field { 
                    display: flex; 
                    align-items: center; 
                    margin-bottom: 16px;
                  }
                  .field label { 
                    font-weight: bold; 
                    width: 100px; 
                    line-height: 1;
                  }
                  .field span { 
                    border-bottom: 1px solid #000; 
                    flex-grow: 1; 
                    padding-bottom: 4px;
                  }
                  .field .no-underline { 
                    border-bottom: none;
                  }
                  .signature-section { 
                    text-align: center; 
                    margin: 15px 0; 
                  }
                  .signature { 
                    display: flex; 
                    align-items: center; 
                    margin: 10px 0; 
                  }
                  .signature label { 
                    font-weight: bold; 
                    width: 100px; 
                  }
                  .signature span { 
                    border-bottom: 1px solid #000; 
                    flex-grow: 1; 
                  }
                  .signature-note { 
                    font-size: 10px; 
                    margin-top: 2px; 
                    text-align: center; 
                  }
                  .observation { 
                    font-size: 10px; 
                    margin-top: 15px; 
                  }
                  .local { 
                    text-align: center; 
                    font-size: 10px; 
                    margin-bottom: 16px;
                  }
                </style>
              </head>
              <body>
                <h2>MARINHA DO BRASIL</h2>
                <h3>SERVIÇO DE IDENTIFICAÇÃO DA MARINHA</h3>
                <h4>TERMO DE RECEBIMENTO (2ª Via de Recibo de Caixa)</h4>
                <p class="local">Local: <span>SIM</span></p>

                <div class="field">
                  <label>Data:</label>
                  <span class="no-underline">${dataAtual}</span>
                </div>
                <div class="field">
                  <label>Registro:</label>
                  <span></span>
                </div>
                <div class="field">
                  <label>NIP:</label>
                  <span>${nip}</span>
                </div>
                <div class="field">
                  <label>Nome Completo:</label>
                  <span></span>
                </div>
                <div class="field">
                  <label>Produto:</label>
                  <span>Cartão de Identidade</span>
                </div>

                <div class="signature-section">
                  ${assinatura}
                </div>

                <div class="signature">
                  <label>Recebido por:</label>
                  <span style="border-bottom: 1px solid #000; flex-grow: 1;"></span>
                  <div class="signature-note">
                    <span style="border-top: 1px solid #000; display: block; border-bottom: none;">(Assinatura idêntica à que consta no Cartão de Identidade)</span>
                  </div>
                </div>

                <div class="signature" style="margin-top: 28px;">
                  <label>Autorizado por:</label>
                  <span style="border-bottom: 1px solid #000; flex-grow: 1;"></span>
                  <div class="signature-note">
                    <span style="border-top: 1px solid #000; display: block; border-bottom: none;">(Assinatura idêntica à que consta no Cartão de Identidade)</span>
                  </div>
                </div>

                <div class="observation">
                  <strong>Obs.:</strong> A retirada do Cartão de Identidade (CI) por terceiros deverá ser autorizada pelo identificado, conforme mencionado acima. Sessenta dias após a sua emissão, o documento não retirado será destruído. Para verificar a disponibilidade, favor ligar para o telefone (21) 2104-6629, ramal 4300, ou acessar o site simweb.sim.mb/Mol/.
                </div>
              </body>
            </html>
          `;

          const novaJanela = window.open("", "_blank");
          novaJanela.document.write(reciboHTML);
          novaJanela.document.close();
          return;
        }
      }

      alert("NIP não encontrado.");
    }

    function salvarEPular() {
      const nip = getUltimoNIPInserido();
      if (nip) {
        const nipElemento = document.querySelector(`[id^="nip-${nip}-"]`);
        if (nipElemento) {
          nipElemento.scrollIntoView({ behavior: "smooth", block: "center" });
        }
      }
    }

    function getUltimoNIPInserido() {
      const tabela = document.getElementById("tabela").getElementsByTagName("tbody")[0];
      if (tabela.rows.length === 0) return null;
      return tabela.rows[tabela.rows.length - 1].cells[0].textContent;
    }
  </script>

</body>
</html>
