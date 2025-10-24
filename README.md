# Article Translator (Azure OpenAI + File Picker / URL)

Tradutor de artigos/documentos usando **Azure OpenAI** via `langchain-openai`.
Suporta **upload de arquivo** (TXT, MD, HTML/HTM, PDF, DOCX) ou **URL**. Inclui *chunking* para textos longos e opção de **salvar a tradução** em arquivo.

---

## 🎯 Recursos
- Azure OpenAI com `AzureChatOpenAI` (deployment ex.: `gpt-4o-mini`)
- Upload de arquivo (via `ipywidgets`) **ou** entrada por URL
- Extração de texto de: TXT/MD, HTML/HTM, PDF (`pdfminer.six`), DOCX (`python-docx`)
- Chunking de texto para evitar limites de token
- Salvar tradução em `translated/<nome>_<idioma>.txt`
- Fallback sem UI (caso `ipywidgets` não esteja disponível)

---

## 📦 Requisitos

```bash
pip install -U langchain-openai openai requests beautifulsoup4 pdfminer.six python-docx ipywidgets
# Se usar Jupyter Notebook clássico:
jupyter nbextension enable --py widgetsnbextension
# (JupyterLab não precisa do comando de enable)
```

> No VSCode: instale a extensão **Jupyter** para abrir/rodar `.ipynb` com widgets.

---

## 🔐 Configuração do Azure OpenAI

Você pode definir as variáveis **via `.env`** (recomendado) ou diretamente no sistema.

### Opção A) Usando `.env` (recomendado)
1. Instale:
   ```bash
   pip install python-dotenv
   ```
2. Crie um arquivo `.env` na raiz do projeto:
   ```ini
   AZURE_OPENAI_ENDPOINT=https://SEU-RECURSO.openai.azure.com/
   AZURE_OPENAI_API_KEY=SUAS-CHAVE-AQUI
   AZURE_OPENAI_API_VERSION=2024-02-15-preview
   AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
   ```
3. No código Python (logo no início), carregue o `.env`:
   ```python
   from dotenv import load_dotenv
   load_dotenv()  # carrega o .env para o ambiente
   ```
4. Depois, leia com `os.getenv()`:
   ```python
   import os
   AZURE_OPENAI_ENDPOINT = os.getenv("AZURE_OPENAI_ENDPOINT")
   AZURE_OPENAI_API_KEY  = os.getenv("AZURE_OPENAI_API_KEY")
   AZURE_OPENAI_API_VER  = os.getenv("AZURE_OPENAI_API_VERSION", "2024-02-15-preview")
   AZURE_DEPLOYMENT_NAME = os.getenv("AZURE_OPENAI_DEPLOYMENT", "gpt-4o-mini")
   ```

> `os.getenv()` **lê variáveis do ambiente do processo**. O `.env` **não é carregado automaticamente**; por isso usamos `python-dotenv` e `load_dotenv()`.

### Opção B) Variáveis de ambiente do sistema

**PowerShell (sessão atual):**
```powershell
$env:AZURE_OPENAI_ENDPOINT = "https://SEU-RECURSO.openai.azure.com/"
$env:AZURE_OPENAI_API_KEY  = "SUA-CHAVE"
$env:AZURE_OPENAI_API_VERSION = "2024-02-15-preview"
$env:AZURE_OPENAI_DEPLOYMENT = "gpt-4o-mini"
```

**PowerShell (persistente):**
```powershell
setx AZURE_OPENAI_ENDPOINT "https://SEU-RECURSO.openai.azure.com/"
setx AZURE_OPENAI_API_KEY "SUA-CHAVE"
setx AZURE_OPENAI_API_VERSION "2024-02-15-preview"
setx AZURE_OPENAI_DEPLOYMENT "gpt-4o-mini"
# reinicie o terminal/VSCode para surtir efeito
```

**CMD (sessão):**
```bat
set AZURE_OPENAI_ENDPOINT=https://SEU-RECURSO.openai.azure.com/
set AZURE_OPENAI_API_KEY=SUA-CHAVE
set AZURE_OPENAI_API_VERSION=2024-02-15-preview
set AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

**Bash (Linux/macOS):**
```bash
export AZURE_OPENAI_ENDPOINT="https://SEU-RECURSO.openai.azure.com/"
export AZURE_OPENAI_API_KEY="SUA-CHAVE"
export AZURE_OPENAI_API_VERSION="2024-02-15-preview"
export AZURE_OPENAI_DEPLOYMENT="gpt-4o-mini"
```

---

## ▶️ Uso

### 1) Via interface (Jupyter/VSCode com widgets)
- Abra o notebook.
- As caixas de **upload** e de **URL** aparecerão.
- Escolha o **arquivo** ou informe uma **URL**, defina o idioma (ex.: `pt-BR`) e clique em **Traduzir**.
- A tradução pode ser salva automaticamente em `translated/`.

### 2) Sem widgets (funções diretas)
```python
from pathlib import Path

texto = extract_text_from_file(Path("meu_arquivo.pdf"))  # ou .txt/.md/.html/.docx/.pdf
trad = translate_text(texto, "pt-BR")
print(trad[:1000])

out = save_translation(trad, "pt-BR", "meu_arquivo.pdf")
print("Arquivo salvo em:", out)
```

Ou por URL:
```python
texto = extract_text_from_url("https://exemplo.com/artigo")
trad = translate_text(texto, "pt-BR")
```

---

## 🧠 Cliente Azure (LangChain)

Exemplo mínimo do cliente exigido:
```python
from langchain_openai import AzureChatOpenAI

client = AzureChatOpenAI(
    azure_endpoint=AZURE_OPENAI_ENDPOINT,
    api_key=AZURE_OPENAI_API_KEY,
    api_version=AZURE_OPENAI_API_VER,
    deployment_name=AZURE_DEPLOYMENT_NAME,
    max_retries=0,
    temperature=0.2,
)
```

Chamada:
```python
resp = client.invoke([
    {"role": "system", "content": "You are a professional translator."},
    {"role": "user", "content": "Diga apenas 'ok'."}
])
print(resp.content)
```

---

## 🧩 Estrutura sugerida

```
.
├── articleTranslator.ipynb                 # original (se aplicável)
├── articleTranslator_file_picker.ipynb     # versão com seletor de arquivo
├── translated/                             # saídas salvas
├── uploads/                                # arquivos enviados via UI
├── .env                                    # credenciais Azure (opcional, via python-dotenv)
└── README.md
```

---

## ❗️ Solução de problemas

- **`The term 'jupyter' is not recognized`**: Jupyter não instalado/fora do PATH.
  ```bash
  pip install notebook  # ou jupyterlab
  jupyter --version
  ```

- **Widgets não aparecem**: instale `ipywidgets` e ative para Notebook clássico:
  ```bash
  pip install ipywidgets
  jupyter nbextension enable --py widgetsnbextension
  ```
  (JupyterLab não precisa do enable.)

- **`OpenAIError: Missing credentials`**: verifique `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_API_VERSION`, `AZURE_OPENAI_DEPLOYMENT`.
  - Confirme `deployment_name` = **nome do deployment** no Azure (não o nome do modelo).
  - Cheque a `api_version` suportada pelo seu recurso.

- **PDF/DOCX não extraem**: garanta `pdfminer.six` e `python-docx` instalados.

---

## 📝 Licença
Use livremente para estudos e projetos internos.
