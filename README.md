# 🖥️ Zabbix Proxy Monitor Card — Neumorphism Dark

> Card de monitoramento visual para painéis **Zabbix + Grafana (HTML Graphics)**, com design Neumorphism Dark, anéis de progresso SVG, alertas por cor e suporte a logos customizadas por cliente.

![Preview](https://img.shields.io/badge/Zabbix-6.x%20%7C%207.x-red?style=flat-square&logo=zabbix)
![MariaDB](https://img.shields.io/badge/MariaDB-compatível-blue?style=flat-square&logo=mariadb)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

---

## 📸 Preview
ON
<img width="363" height="177" alt="image" src="https://github.com/user-attachments/assets/d470d4ce-845e-4782-a767-8b6983c01c86" />
OF
<img width="417" height="187" alt="image" src="https://github.com/user-attachments/assets/a1a891d2-598e-46ce-b6fc-4f108890f207" />


---

## 📁 Estrutura do repositório

```
zabbix-proxy-card/
├── htmlPx.txt           # HTML/SVG do card (campo "HTML/SVG document")
├── cssPx.txt            # CSS base do painel Grafana (campo "CSS")
├── jsPx-onrender.txt    # JavaScript do card (campo "onRender")
├── querymysql.txt       # Query MySQL/MariaDB para o Zabbix
└── README.md            # Este arquivo
```

---

## ⚙️ Itens monitorados

| Métrica | Item Key Zabbix | Tipo | Tabela |
|---|---|---|---|
| CPU | `system.cpu.util` | Float | `history` |
| Memória | `vm.memory.utilization` | Float | `history` |
| Disco `/` | `vfs.fs.size[/,pused]` | Float | `history` |
| Processos ativos | `proc.num[,,run]` | Unsigned | `history_uint` |
| Agent Ping | `agent.ping` | Unsigned | `history_uint` |

---

## 🎨 Regras de cor por criticidade

### CPU · RAM · Disco
| Faixa | Cor |
|---|---|
| 0% – 50% | 🟢 Verde `#4ade80` |
| 51% – 79% | 🟡 Amarelo `#fbbf24` |
| 80% – 100% | 🔴 Vermelho `#ef4444` |

### Processos ativos
| Faixa | Cor |
|---|---|
| 0 – 15 | 🟢 Verde `#4ade80` |
| 16 – 29 | 🟡 Amarelo `#fbbf24` |
| 30+ | 🔴 Vermelho `#ef4444` |

### Badge de status (agent.ping)
| Valor | Badge | Cor |
|---|---|---|
| `1` (online) | `PRX-ON` | 🟢 Verde pulsante |
| `0` (offline) | `PRX-OFF` | 🔴 Vermelho estático |

---

## 🚀 Como instalar

### 1. Pré-requisitos
- Zabbix 6.x ou 7.x com agente ativo no host
- Grafana com plugin **HTML Graphics** instalado
- Datasource MySQL/MariaDB apontando para o banco do Zabbix

### 2. Configurar a Query MySQL

Abra `querymysql.txt` e substitua `NOME_HOST` pelo hostname exato do host no Zabbix:

```sql
AND h.host = 'NOME-DO-HOST'   -- substitua pelo seu hostname
```

Cole a query no campo **Query A** do painel HTML Graphics no Grafana.

> ⚠️ **Atenção:** cole apenas **uma query por vez** no campo. Múltiplas queries causam erro 1064 no MariaDB.

### 3. Configurar o CSS do painel

No painel HTML Graphics, role até o campo **CSS** e cole o conteúdo de `cssPx.txt`:

```css
* {
  font-family: Open Sans;
}
```

### 4. Configurar o HTML/SVG document

Cole o conteúdo de `htmlPx.txt` no campo **HTML/SVG document** do painel.

Para trocar a **logo do cliente**, localize este trecho e substitua a URL:

```html
<!-- Logo do cliente -->
<div class="logo-box">
  <img
    src="https://SUA-URL-AQUI/logo-cliente.png"
    alt="Cliente"
    onerror="this.style.opacity='0.2';"
  />
</div>
```

> A logo do Zabbix fica sempre à direita da logo do cliente. Ambas possuem fallback automático caso a URL não carregue.

### 5. Configurar o onRender

No painel HTML Graphics, role até o campo **onRender** e cole o conteúdo de `jsPx-onrender.txt`.

> ⚠️ **Importante:** cole apenas o **corpo** do código, **sem** `function onRender() { }`. O Grafana injeta isso automaticamente. Colar a declaração da função causa erro `Unexpected token ')'`.

Ative a opção **"Run onRender when mounted"**.

---

## 🔌 Colunas retornadas pela Query

O `onRender` lê as seguintes colunas pelo nome exato. Não altere os aliases da query:

| Alias SQL | Variável JS | Uso |
|---|---|---|
| `hostname` | `host` | Nome exibido no card |
| `cpu_pct` | `cpu` | Anel CPU |
| `ram_pct` | `ram` | Anel RAM |
| `disk_pct` | `disk` | Anel Disco |
| `proc_running` | `proc` | Número de processos |
| `agent_ping` | `ping` | Status PRX-ON / PRX-OFF |

---

## 🛠️ Personalização

### Trocar o hostname no fallback JS

No arquivo `jsPx-onrender.txt`, linha:
```js
var host = getVal('hostname') || 'NOME-DO-HOST';
```
Substitua `'NOME-DO-HOST'` pelo hostname padrão desejado.

### Ajustar os limiares de cor

No arquivo `jsPx-onrender.txt`, edite as funções:

```js
// CPU, RAM, Disco
function colorPct(pct) {
  if (pct >= 80) return '#ef4444';  // vermelho
  if (pct >= 51) return '#fbbf24';  // amarelo
  return '#4ade80';                 // verde
}

// Processos
function colorProc(val) {
  if (val >= 30) return '#ef4444';  // vermelho
  if (val >= 16) return '#fbbf24';  // amarelo
  return '#4ade80';                 // verde
}
```

### Adicionar mais hosts

Duplique o painel no Grafana e altere apenas o `AND h.host = '...'` na query de cada painel.

---

## 🐛 Troubleshooting

| Erro | Causa | Solução |
|---|---|---|
| `Error 1064 (42000)` | Duas queries no mesmo campo | Cole apenas uma query por painel |
| `Error 1054: Unknown column 'i.lastvalue'` | Zabbix 6+ removeu essa coluna | Use a query de `querymysql.txt` que lê de `history` diretamente |
| `Error executing onRender: Unexpected token ')'` | Declaração `function onRender(){}` colada no campo | Cole apenas o corpo, sem a declaração da função |
| Card exibe `—` em todos os campos | `onRender` não lê os dados da query | Confirme que as colunas têm os aliases exatos listados acima |
| Logo do cliente não aparece | URL bloqueada pela rede | Hospede a imagem internamente ou use base64 |

---

## 🧩 Compatibilidade

| Componente | Versão testada |
|---|---|
| Zabbix | 6.x, 7.x |
| Grafana | 9.x, 10.x |
| Plugin HTML Graphics | qualquer versão atual |
| Banco de dados | MariaDB (MySQL compatível) |
