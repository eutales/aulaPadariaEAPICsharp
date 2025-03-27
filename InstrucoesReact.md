Para criar um site em React que consuma a API de cadastro de produtos da padaria (a mesma que você criou em C#), vamos dividir o processo em várias etapas.

### Passo 1: Criando o Projeto React

1. **Instalar o Node.js e o npm**: Antes de começar, verifique se o Node.js e o npm estão instalados em seu sistema. Você pode verificar executando os seguintes comandos no terminal:

   ```bash
   node -v
   npm -v
   ```

   Se não tiver o Node.js instalado, baixe-o [aqui](https://nodejs.org/en/).

2. **Criar um Novo Projeto React**: Abra o terminal, navegue até o diretório onde você deseja criar o projeto e execute o seguinte comando:

   ```bash
   npx create-react-app padaria-frontend
   ```

   Isso criará uma nova pasta chamada `padaria-frontend` com um projeto básico React.

3. **Instalar Dependências Adicionais**: Dentro do diretório do projeto, você precisará instalar a biblioteca `axios` para facilitar a comunicação com a API. Execute o seguinte comando:

   ```bash
   cd padaria-frontend
   npm install axios
   ```

### Passo 2: Criando a Estrutura do Projeto

Dentro do projeto React, você precisará de alguns componentes para manipular a lista de produtos, cadastrar, editar e excluir produtos.

#### Estrutura do Projeto

A estrutura do projeto será algo como:

```
padaria-frontend/
├── public/
├── src/
│   ├── components/
│   │   ├── ProdutoForm.js
│   │   ├── ProdutoList.js
│   ├── App.js
│   ├── App.css
│   ├── index.js
├── package.json
```

### Passo 3: Criando os Componentes React

#### 1. **ProdutoForm.js**

Este componente será responsável por exibir o formulário de cadastro e edição de produtos.

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const ProdutoForm = ({ produto, onSave }) => {
  const [nome, setNome] = useState('');
  const [descricao, setDescricao] = useState('');
  const [preco, setPreco] = useState('');
  const [quantidade, setQuantidade] = useState('');

  useEffect(() => {
    if (produto) {
      setNome(produto.nome);
      setDescricao(produto.descricao);
      setPreco(produto.preco);
      setQuantidade(produto.quantidade);
    }
  }, [produto]);

  const handleSubmit = async (e) => {
    e.preventDefault();
    const novoProduto = { nome, descricao, preco, quantidade };

    if (produto) {
      // Editar produto
      await axios.put(`http://localhost:5000/api/produtos/${produto.id}`, novoProduto);
    } else {
      // Adicionar novo produto
      await axios.post('http://localhost:5000/api/produtos', novoProduto);
    }

    onSave(); // Atualiza a lista após salvar
    setNome('');
    setDescricao('');
    setPreco('');
    setQuantidade('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>{produto ? 'Editar Produto' : 'Adicionar Produto'}</h2>
      <div>
        <label>Nome</label>
        <input
          type="text"
          value={nome}
          onChange={(e) => setNome(e.target.value)}
          required
        />
      </div>
      <div>
        <label>Descrição</label>
        <textarea
          value={descricao}
          onChange={(e) => setDescricao(e.target.value)}
        />
      </div>
      <div>
        <label>Preço</label>
        <input
          type="number"
          value={preco}
          onChange={(e) => setPreco(e.target.value)}
          required
        />
      </div>
      <div>
        <label>Quantidade</label>
        <input
          type="number"
          value={quantidade}
          onChange={(e) => setQuantidade(e.target.value)}
          required
        />
      </div>
      <button type="submit">{produto ? 'Atualizar' : 'Adicionar'}</button>
    </form>
  );
};

export default ProdutoForm;
```

#### 2. **ProdutoList.js**

Este componente será responsável por exibir a lista de produtos e permitir a exclusão e edição.

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import ProdutoForm from './ProdutoForm';

const ProdutoList = () => {
  const [produtos, setProdutos] = useState([]);
  const [produtoEditando, setProdutoEditando] = useState(null);

  useEffect(() => {
    // Carregar a lista de produtos
    const carregarProdutos = async () => {
      const response = await axios.get('http://localhost:5000/api/produtos');
      setProdutos(response.data);
    };

    carregarProdutos();
  }, []);

  const handleDelete = async (id) => {
    await axios.delete(`http://localhost:5000/api/produtos/${id}`);
    setProdutos(produtos.filter((produto) => produto.id !== id));
  };

  const handleSave = () => {
    // Recarregar a lista de produtos após salvar
    axios.get('http://localhost:5000/api/produtos')
      .then(response => setProdutos(response.data));
    setProdutoEditando(null);
  };

  return (
    <div>
      <h1>Produtos</h1>
      <ProdutoForm produto={produtoEditando} onSave={handleSave} />

      <table>
        <thead>
          <tr>
            <th>Nome</th>
            <th>Preço</th>
            <th>Quantidade</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          {produtos.map((produto) => (
            <tr key={produto.id}>
              <td>{produto.nome}</td>
              <td>{produto.preco}</td>
              <td>{produto.quantidade}</td>
              <td>
                <button onClick={() => setProdutoEditando(produto)}>Editar</button>
                <button onClick={() => handleDelete(produto.id)}>Excluir</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

export default ProdutoList;
```

#### 3. **App.js**

Agora, no arquivo `App.js`, importe e use o componente `ProdutoList` que contém a lista de produtos e o formulário de cadastro/edição:

```javascript
import React from 'react';
import ProdutoList from './components/ProdutoList';
import './App.css';

const App = () => {
  return (
    <div className="App">
      <ProdutoList />
    </div>
  );
};

export default App;
```

#### 4. **App.css (opcional)**

Você pode adicionar alguns estilos básicos para deixar a interface mais organizada. Adicione este código no arquivo `App.css`:

```css
.App {
  font-family: Arial, sans-serif;
  padding: 20px;
}

form {
  margin-bottom: 20px;
}

form div {
  margin-bottom: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

table, th, td {
  border: 1px solid #ddd;
}

th, td {
  padding: 8px;
  text-align: left;
}

button {
  padding: 6px 12px;
  margin: 5px;
}
```

### Passo 4: Rodando o Projeto React

1. **Executar o Backend**: Verifique se a sua API C# (com XAMPP) está em execução no `http://localhost:5000`.

2. **Rodar o Frontend React**: No terminal, dentro do diretório do seu projeto React, execute o seguinte comando para iniciar o servidor de desenvolvimento do React:

   ```bash
   npm start
   ```

   Isso abrirá o seu site no navegador na URL `http://localhost:3000`.

### Passo 5: Testando a Aplicação

Agora, você pode testar o site em React, que deve:

1. Exibir a lista de produtos da padaria.
2. Permitir que você adicione novos produtos.
3. Permitir que você edite produtos existentes.
4. Permitir que você exclua produtos.

### Conclusão

Você criou um site em React que consome a API criada com C# para gerenciar o cadastro de produtos de uma padaria. Isso inclui as operações CRUD (Criar, Ler, Atualizar, Excluir). O site utiliza `axios` para fazer as requisições HTTP à API e exibe as informações de forma interativa.
