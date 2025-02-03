# Criando um Web App no Azure para Validar CPF

Este guia descreve como criar um Web App no Azure que valida números de CPF usando uma API simples.

## 1. Criar uma Conta no Azure
Se você ainda não possui uma conta no Azure, crie uma em [https://azure.microsoft.com](https://azure.microsoft.com).

## 2. Criar um Web App no Azure
1. Acesse o portal do Azure ([https://portal.azure.com](https://portal.azure.com)).
2. No menu esquerdo, clique em **App Services** e depois em **+ Criar**.
3. Preencha os campos:
   - **Nome do Aplicativo**: `validador-cpf`
   - **Grupo de Recursos**: Crie um novo ou use um existente.
   - **Publicação**: Código
   - **Runtime Stack**: Escolha Python ou Node.js conforme a linguagem que deseja usar.
   - **Região**: Escolha uma próxima a você.
4. Clique em **Revisar + Criar** e depois em **Criar**.

## 3. Criar a API para Validar CPF
A seguir, criaremos uma API simples para validar um número de CPF. 

### Exemplo com Python (Flask)

1. Instale as dependências:
   ```sh
   pip install flask
   ```
2. Crie um arquivo `app.py`:
   ```python
   from flask import Flask, request, jsonify

   app = Flask(__name__)

   def validar_cpf(cpf):
       cpf = ''.join(filter(str.isdigit, cpf))
       if len(cpf) != 11:
           return False
       soma = sum(int(cpf[i]) * (10 - i) for i in range(9))
       d1 = (soma * 10 % 11) % 10
       soma = sum(int(cpf[i]) * (11 - i) for i in range(10))
       d2 = (soma * 10 % 11) % 10
       return d1 == int(cpf[9]) and d2 == int(cpf[10])

   @app.route("/validar", methods=["GET"])
   def validar():
       cpf = request.args.get("cpf")
       if not cpf:
           return jsonify({"erro": "CPF não informado"}), 400
       valido = validar_cpf(cpf)
       return jsonify({"cpf": cpf, "valido": valido})

   if __name__ == "__main__":
       app.run(host="0.0.0.0", port=5000)
   ```
3. Teste localmente:
   ```sh
   python app.py
   ```
   Acesse `http://127.0.0.1:5000/validar?cpf=12345678909`.

## 4. Fazer o Deploy no Azure

### Opção 1: Deploy via GitHub Actions
1. No portal do Azure, vá até o **App Services** e selecione seu aplicativo.
2. No menu lateral, clique em **Deployment Center**.
3. Escolha **GitHub** como origem do código.
4. Conecte sua conta do GitHub e selecione o repositório onde seu código está armazenado.
5. Configure o **Runtime Stack** para coincidir com a linguagem usada (por exemplo, Python 3.9).
6. O Azure criará um workflow do GitHub Actions automaticamente.
7. Após configurar, todo push para o repositório acionará um novo deploy automático no Azure.

### Opção 2: Deploy via FTP
1. No portal do Azure, acesse seu Web App.
2. Vá até **Deployment Center** e selecione **FTP**.
3. Habilite o FTP e copie as credenciais de login fornecidas.
4. Use um cliente FTP (como FileZilla) para se conectar ao servidor.
5. Envie todos os arquivos do projeto para o diretório correto (`/site/wwwroot`).
6. Reinicie o Web App no Azure para que as alterações entrem em vigor.

### Opção 3: Deploy via Azure CLI
1. Instale a Azure CLI se ainda não tiver:
   ```sh
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```
2. Faça login na sua conta do Azure:
   ```sh
   az login
   ```
3. Publique o aplicativo usando o seguinte comando:
   ```sh
   az webapp up --name validador-cpf --resource-group MeuGrupo --runtime "PYTHON:3.9"
   ```
4. Após a conclusão do deploy, a API estará disponível em:
   ```
   https://validador-cpf.azurewebsites.net/validar?cpf=12345678909
   ```

## 5. Testar a API
Utilize ferramentas como Postman ou cURL:
```sh
curl "https://validador-cpf.azurewebsites.net/validar?cpf=12345678909"
```

Agora, seu Web App no Azure está pronto para validar CPFs!

