# sistema-gestao-estoque
Sistema Web para cadastro de produtos, controle de estoque e gerenciamento de fileiras e estantes.
// Sistema de Gestão de Estoque - Node.js + Express + SQLite + HTML

const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const path = require('path');
const app = express();
const PORT = 3000;

// Middleware
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
app.use(express.static('public'));

// Banco de dados
const db = new sqlite3.Database('./estoque.db', (err) => {
  if (err) return console.error(err.message);
  console.log('🗄️  Banco de dados conectado.');
});

// Criação das tabelas
const createTables = () => {
  db.run(`CREATE TABLE IF NOT EXISTS estante (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL
  )`);

  db.run(`CREATE TABLE IF NOT EXISTS fileira (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL,
    estante_id INTEGER,
    FOREIGN KEY(estante_id) REFERENCES estante(id)
  )`);

  db.run(`CREATE TABLE IF NOT EXISTS produto (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL,
    codigo TEXT,
    descricao TEXT,
    quantidade INTEGER,
    fileira_id INTEGER,
    FOREIGN KEY(fileira_id) REFERENCES fileira(id)
  )`);
};

createTables();

// Rotas principais
app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, '/public/index.html'));
});

// Cadastro de estante
app.post('/estantes', (req, res) => {
  const { nome } = req.body;
  db.run('INSERT INTO estante (nome) VALUES (?)', [nome], (err) => {
    if (err) return res.status(500).send(err.message);
    res.redirect('/');
  });
});

// Cadastro de fileira
app.post('/fileiras', (req, res) => {
  const { nome, estante_id } = req.body;
  db.run('INSERT INTO fileira (nome, estante_id) VALUES (?, ?)', [nome, estante_id], (err) => {
    if (err) return res.status(500).send(err.message);
    res.redirect('/');
  });
});

// Cadastro de produto
app.post('/produtos', (req, res) => {
  const { nome, codigo, descricao, quantidade, fileira_id } = req.body;
  db.run(
    'INSERT INTO produto (nome, codigo, descricao, quantidade, fileira_id) VALUES (?, ?, ?, ?, ?)',
    [nome, codigo, descricao, quantidade, fileira_id],
    (err) => {
      if (err) return res.status(500).send(err.message);
      res.redirect('/');
    }
  );
});

// Listagem de produtos (JSON)
app.get('/produtos', (req, res) => {
  db.all(
    `SELECT p.id, p.nome, p.codigo, p.descricao, p.quantidade, f.nome AS fileira, e.nome AS estante
     FROM produto p
     JOIN fileira f ON p.fileira_id = f.id
     JOIN estante e ON f.estante_id = e.id`,
    [],
    (err, rows) => {
      if (err) return res.status(500).send(err.message);
      res.json(rows);
    }
  );
});

// Inicia servidor
app.listen(PORT, () => {
  console.log(`🚀 Servidor rodando em http://localhost:${PORT}`);
});
