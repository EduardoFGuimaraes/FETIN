# 📋 Guia — Rodando o AeroScan no MacBook com GPS do celular (dia da feira)

Roteiro para apresentar a detecção de focos ao vivo, usando o GPS real de um
celular (via rede) em vez de um módulo GPS físico.

Os dois caminhos já foram testados na prática:

| Celular | App | Porta | Situação |
|---------|-----|-------|----------|
| **Android** | GPS Tether Server | varia | testado |
| **iPhone** | GPS2IP / GPS2IP Lite | `11123` | testado (ver passo 4b) |

O que o script precisa é sempre o mesmo: um app que transmita as sentenças
NMEA do GPS por **TCP**, e o celular alcançável pelo Mac na rede.

---

## 1. Montar a rede

O script só precisa que o Mac **alcance o celular por IP**. Existem duas
formas, e a mais simples costuma bastar:

### Wi-Fi comum (use esta quando der)

Coloque o MacBook e o celular na **mesma rede Wi-Fi** e pronto — não precisa
de hotspot, nem de internet. Foi assim que o teste completo rodou: Mac,
iPhone e tablet numa rede doméstica, tudo funcionando.

### Hotspot (só se a rede do local bloquear)

Redes de eventos/feiras costumam ter "isolamento de cliente" ativado, que
impede um celular de falar diretamente com um notebook mesmo estando na
mesma rede. **Se e somente se** isso acontecer, use um hotspot pessoal só
para os aparelhos da demonstração:

1. No celular que vai virar o hotspot: ative o **Compartilhamento de
   Internet / Hotspot Pessoal** e defina uma senha.
2. No MacBook: conecte no Wi-Fi desse hotspot.
3. Se o celular do GPS for outro aparelho, conecte ele no mesmo hotspot.

O mesmo celular pode ser hotspot **e** fonte de GPS ao mesmo tempo — o chip
de GPS é receptor de satélite e não depende dos dados móveis. Quando o
Android é o ponto de acesso, o IP dele costuma ser `192.168.43.1`; no
iPhone, `172.20.10.1`.

> ⚠️ Um tablet ou aparelho **sem chip** não consegue criar hotspot (Android e
> iPad exigem conexão celular para isso). Ele entra na rede só como cliente.

> ⚠️ O IP do celular muda toda vez que a rede muda — depois de conectar,
> pegue o IP novo (passo 4), não reaproveite um IP de um teste anterior em
> outra rede.

---

## 2. Preparar o ambiente Python no MacBook

Fazer **com antecedência**, não no dia da apresentação:

**Antes de tudo, confira a versão do Python.** O `python3` que vem com as
ferramentas de linha de comando da Apple é o 3.9, e o `requirements.txt`
não instala nele: o `pandas==3.0.5` exige 3.11 ou mais novo, e o
`roboflow==1.4.0` exige 3.10 ou mais novo. Rodar `python3 -m venv` num Mac
de fábrica cria um ambiente 3.9 e o `pip install` quebra no meio.

```bash
python3 --version
```

Se der **3.10 ou menor**, instale um Python mais novo antes de continuar
(via [Homebrew](https://brew.sh)) e use ele no lugar do `python3`:

```bash
brew install python@3.12
```

Com o Python certo em mãos:

```bash
git clone https://github.com/1matheeus/FETIN.git
cd FETIN
python3.12 -m venv venv          # ou o python3.XX que você tiver, 3.11+
source venv/bin/activate
pip install -r requirements.txt
```

- Confirme que o venv nasceu na versão certa antes de instalar:
  `python --version` (já com o venv ativado) tem que mostrar 3.11 ou mais.
- `source venv/bin/activate` é o equivalente do `venv\Scripts\activate` do
  Windows — precisa rodar isso toda vez que abrir um Terminal novo, antes
  dos comandos dos passos seguintes.
- Se der erro instalando `ultralytics` ou `opencv-python` (comum em Macs
  com chip Apple Silicon — M1/M2/M3), guarde a mensagem de erro para
  resolver com calma antes do dia da feira.

> Testado num MacBook Apple Silicon com Python 3.12.14: as 59 dependências
> instalam sem erro, o modelo carrega e o PyTorch reconhece a GPU do Mac
> (MPS).

## 3. Testar a câmera uma vez (webcam do Mac)

```bash
python demo.py
```

Na **primeira vez**, o macOS pergunta "Terminal gostaria de acessar a
câmera" — clique em **Permitir**. Teste isso com antecedência — não é bom
descobrir esse popup no meio da demonstração.

**Se a permissão já tiver sido negada** (ou o popup passou batido), o script
não avisa que o problema é permissão: ele varre os índices de câmera, nenhum
abre, e ele encerra com `Nenhuma câmera encontrada.` — como se o Mac não
tivesse câmera nenhuma. No Terminal aparece:

```
OpenCV: not authorized to capture video (status 0), requesting...
OpenCV: camera failed to properly initialize!
Nenhuma câmera encontrada.
```

Para resolver:

1. **Ajustes do Sistema > Privacidade e Segurança > Câmera**
2. Ligue o app que você usa para rodar o comando
3. **Feche e reabra o app** — a permissão só passa a valer depois disso

> ⚠️ A permissão é do **aplicativo que roda o Python**, não do Python. Se
> você testou pelo VS Code e no dia da feira vai abrir o Terminal, são duas
> autorizações diferentes — libere a do app que você vai realmente usar na
> apresentação.

Se o Mac tiver mais de uma câmera disponível (a FaceTime HD embutida e o
iPhone via Continuidade, por exemplo), o script pega **a primeira que
abrir** — que nem sempre é a que você quer. Confira qual entrou antes de
apresentar; o `drone/detectar_foco.py` aceita `--camera 1` para escolher
outra.

## 4. Iniciar o servidor de GPS no celular

Vale para os dois sistemas: **conceda a permissão de localização**, escolha
**TCP** (não UDP/broadcast) se o app perguntar, e anote o **IP e a porta**
que ele mostra na tela.

> 🛰️ **Fique perto de uma janela ou saia na rua.** O GPS precisa enxergar
> satélites. Dentro do prédio ele pode nunca fixar, e aí não adianta app
> nenhum — o problema é o sinal, não o software.

### 4a. Android — GPS Tether Server

1. Abra o app e conceda a localização ("permitir sempre" se for oferecido).
2. Escolha o modo **TCP**.
3. Ative/inicie o servidor e anote IP e porta.

### 4b. iPhone — GPS2IP (ou GPS2IP Lite)

1. Instale o **GPS2IP** na App Store (a versão **Lite** é gratuita e serve).
2. Abra e conceda a permissão de localização.
3. Escolha **TCP** como modo de saída.
4. Ative o servidor. A porta padrão é **`11123`** — a mesma que aparece nos
   exemplos do README.
5. O app mostra o IP do iPhone na rede; é ele que vai no `--gps-rede`.

> ⚠️ **Mantenha a tela do iPhone acesa e o app em primeiro plano.** Com a
> tela bloqueada o iOS suspende o app e a transmissão para. No teste, a
> conexão caiu uma vez com `No route to host` e voltou sozinha na tentativa
> seguinte.
>
> **Isso importa mais do que parece:** se a conexão cair com o script já
> rodando, ele **não reconecta e não avisa** — continua carimbando cada novo
> foco com a última coordenada que recebeu, por mais antiga que seja
> (verificado em teste: a leitura morre em silêncio e a posição fica
> congelada). Numa bancada parada isso é inofensivo, porque a coordenada
> certa é sempre a mesma. Andando com o notebook, não é. Se desconfiar que
> caiu, feche com **Q** e rode de novo em vez de seguir.

> ⚠️ Em iPhones com Dynamic Island (ex.: 15 Pro), o botão de ativar o
> servidor pode ficar visualmente atrás dos ícones do sistema. Nesse caso,
> use `Ajustes > Acessibilidade > Controle por Voz > "Mostrar números"` para
> tocar no botão por comando de voz em vez de mirar visualmente.

### 4c. Conferir a conexão antes de rodar a detecção

Vale muito a pena testar a rede **antes** de envolver câmera e modelo — se
algo falhar, você sabe exatamente onde. Trocando `IP` e `PORTA` pelos seus:

```bash
ping -c 3 IP            # o celular responde?
nc -z -v IP PORTA       # a porta está aberta?
```

Como ler o resultado:

| Resposta | Significa |
|----------|-----------|
| `Connection refused` | O celular está na rede, mas **o app não está transmitindo** — ligue o servidor |
| Trava e dá timeout | Firewall ou isolamento de cliente na rede — troque para hotspot (passo 1) |
| `No route to host` | O celular sumiu da rede — tela bloqueada ou Wi-Fi caiu |
| `succeeded!` | Pode seguir |

Para ver as coordenadas cruas chegando (`Ctrl+C` para sair):

```bash
nc IP PORTA
```

Devem aparecer linhas assim, uma ou duas por segundo:

```
$GPGGA,021714,2215.12126,S,04541.68445,W,1,8,0.9,833.1,M,-2.4,M,0,2*60
$GPRMC,021714,A,2215.12126,S,04541.68445,W,0.00,0.00,230926,003.1,W*69
```

Os campos que importam, na `$GPGGA`: depois da longitude vem o **`1`** (fix
válido — se for `0`, o GPS ainda não fixou), o **`8`** (satélites) e o
**`0.9`** (HDOP; abaixo de 1 é ótimo, ~2-5 m). Na `$GPRMC`, o **`A`**
confirma dados válidos — um `V` no lugar significa posição inválida.

## 5. Rodar a detecção de verdade

```bash
python drone/detectar_foco.py --gps-rede IP_DO_CELULAR:PORTA --sem-rede
```

No iPhone com GPS2IP, a porta é a padrão — fica assim:

```bash
python drone/detectar_foco.py --gps-rede 192.168.1.42:11123 --sem-rede
```

- `--gps-rede` conecta no celular e lê o GPS real dele (precisão de
  satélite, ~5-20 m) — é a fonte de localização mais precisa que o projeto
  tem, bem melhor que a localização por Wi-Fi do sistema operacional
  (~1-2 km, pode até cair no bairro errado numa cidade pequena).
- `--sem-rede` evita que o script também tente a localização por Wi-Fi do
  macOS como respaldo — sem isso, na primeira execução o macOS pode abrir
  um popup pedindo permissão de localização para o Terminal, o que
  interromperia a demonstração. Como já temos o GPS do celular, esse
  respaldo não é necessário. Serve também como rede de segurança: se o GPS
  do celular falhar, o script **avisa** em vez de gravar silenciosamente uma
  coordenada ruim.
- Aponte a câmera para a piscina/pneu por ~3 segundos para confirmar um
  foco (o tempo de confirmação existe para evitar falsos positivos de um
  único frame).

### Se nada for confirmado

O padrão exige confiança **≥ 0,60 mantida por 3 segundos**. Filmar uma tela
(tablet, monitor) derruba a confiança por causa de reflexo e brilho. Se o
contador nunca fechar, baixe o limiar:

```bash
python drone/detectar_foco.py --gps-rede IP:PORTA --sem-rede --conf 0.35
```

### Mostrar a imagem de teste em outro aparelho

Para ensaiar sem uma piscina de verdade na frente, dá para servir as imagens
do repositório pela rede local — funciona mesmo sem internet, e é útil quando
o aparelho que vai exibir a foto não tem chip:

```bash
python -m http.server 8000 --bind 0.0.0.0   # na pasta do projeto
```

No outro aparelho, abra `http://IP_DO_MAC:8000/demo_result.jpg` (o IP do Mac
sai de `ipconfig getifaddr en0`). O mesmo servidor também entrega o painel em
`http://IP_DO_MAC:8000/dashboard/index.html` — e servido assim o `fetch()`
dos CSV funciona de verdade, o que não acontece abrindo o arquivo com duplo
clique.

> As duas imagens do repositório (`demo_result.jpg` e `demo_result_tire.jpg`)
> já vêm com as caixas de detecção desenhadas por cima. Servem para ensaiar,
> mas para a apresentação em si vale usar uma imagem limpa.

## 6. Encerrar e levar para o dashboard

1. Aperte **Q** para fechar a janela da câmera.
2. O CSV e as fotos ficam na Área de Trabalho do Mac
   (`~/Desktop/focos_detectados_mvp.csv` e as fotos `foco_<tipo>_<data>.jpg`
   — todas já passaram pelo borrão de rostos do LGPD antes de serem
   salvas).
3. Abra o dashboard (https://1matheeus.github.io/FETIN/dashboard/index.html
   ou local), vá em **Mapa de risco** e clique em **"Importar focos"** —
   selecione o CSV e as fotos da Área de Trabalho juntos. O dashboard casa
   cada foco com a foto pelo nome do arquivo e coloca o marcador no mapa.

---

## Referência rápida de comandos

```bash
# uma vez, antes da feira
git clone https://github.com/1matheeus/FETIN.git
cd FETIN
python3.12 -m venv venv      # precisa ser 3.11+; o 3.9 de fábrica do Mac não serve
source venv/bin/activate
pip install -r requirements.txt
# e libere a câmera para o app que você vai usar:
# Ajustes do Sistema > Privacidade e Segurança > Câmera (depois reabra o app)

# no dia, com Mac e celular na mesma rede e o app de GPS transmitindo
ping -c 3 IP_DO_CELULAR                    # o celular responde?
nc -z -v IP_DO_CELULAR PORTA               # a porta está aberta?
nc IP_DO_CELULAR PORTA                     # chegam linhas $GPGGA/$GPRMC?

source venv/bin/activate
python drone/detectar_foco.py --gps-rede IP_DO_CELULAR:PORTA --sem-rede
# iPhone/GPS2IP usa a porta 11123
# se nada confirmar, acrescente --conf 0.35
```
