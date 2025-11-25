# AI-OFFLINE-LLAMA
<img width="870" height="556" alt="image" src="https://github.com/user-attachments/assets/cf1dc193-1af2-46d1-8eee-5e959424d8f5" />

## Download model LlaMA 3-8B
```bash
https://huggingface.co/QuantFactory/Meta-Llama-3-8B-Instruct-GGUF/blob/main/Meta-Llama-3-8B-Instruct.Q4_K_M.gguf
```

## Download docker image on docker desktop
```bash
docker pull ghcr.io/ggerganov/llama.cpp:server
docker save ghcr.io/ggerganov/llama.cpp:server -o C:\llama-server.tar
```
## Copy model and image on server linux (I`m using Oracle Linux 8)

## Machine requirements
### CPU = 12
### RAM = 32GB
### HARD DISK = 100GB

## Create directory and copy model to directory
```bash
mkdir -p /opt/llama/models
cp ~/model.gguf /opt/llama/models/
```

## Connect to repo offline. Paste file to /etc/yum.repos.d/local.repo
```bash
[docker]
name=Docker CE Stable
baseurl=http://192.168.10.70:8081/repository/docker/
enabled=1
gpgcheck=0

[ol8-baseos]
name=Oracle Linux 8 BaseOS
baseurl=http://192.168.10.70:8081/repository/ol8-baseos/
enabled=1
gpgcheck=0

[ol8-appstream]
name=Oracle Linux 8 AppStream
baseurl=http://192.168.10.70:8081/repository/ol8-appstream/
enabled=1
gpgcheck=0

[ol8-epel]
name=Oracle Linux 8 Extra Packages for Enterprise Linux
baseurl=http://192.168.10.70:8081/repository/ol8-epel/
enabled=1
gpgcheck=0

[ol8-uekr6]
name=Latest Unbreakable Enterprise Kernel Release 6 for Oracle Linux 8
baseurl=http://192.168.10.70:8081/repository/ol8-uekr6/
enabled=1
gpgcheck=0
```

## Install docker and dependencies
```bash
dnf install docker -y
```

## Start docker 
```bash
systemctl enable --now docker.service
systemctl start docker
docker load -i llama.tar
```

## Add port to firewall
```bash
sudo firewall-cmd --permanent --add-port=11434/tcp
sudo firewall-cmd --reload
```

## Run llama container
```bash
docker run -d   --name llama-server   --restart unless-stopped   -p 11434:11434   -v /opt/llama/models:/models   ghcr.io/ggerganov/llama.cpp:server     -m /models/model.gguf     -t 10     --threads 10     --ctx-size 4096     --batch-size 512     --host 0.0.0.0     --port 11434
```

### RUN AI OFFLINE on COMPUTERS
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/c91cdfba-74af-44f7-96d6-9306679afde3" />

## STEP 1. Add port to firewall
```bash
sudo firewall-cmd --permanent --add-port=11434/tcp
sudo firewall-cmd --reload
```

## STEP 2. Add variables to PATH
```bash
export PATH=/usr/local/bin:$PATH
```

## STEP 3. Install python3 and dependencies
```bash
dnf install python3 -y
```

## STEP 4. Create script sgpt on /usr/local/bin/sgpt
```bash
#!/usr/bin/env bash

# Użycie: sgpt "twoje pytanie"
if [ $# -eq 0 ]; then
  echo "Użycie: sgpt \"twoje pytanie\"" >&2
  exit 1
fi

PROMPT="$*"
export PROMPT

python3 - << 'PYTHON'
import os
import json
import sys
from urllib.request import Request, urlopen
from urllib.error import URLError, HTTPError

prompt = os.environ.get("PROMPT", "")
if not prompt:
    print("Błąd: brak PROMPT w środowisku.", file=sys.stderr)
    sys.exit(1)

url = "http://192.168.10.57:11434/v1/chat/completions"
payload = {
    "model": "local",
    "messages": [
        {"role": "user", "content": prompt}
    ]
}

data = json.dumps(payload).encode("utf-8")
req = Request(url, data=data, headers={"Content-Type": "application/json"})

try:
    with urlopen(req) as resp:
        resp_data = resp.read().decode("utf-8")
except HTTPError as e:
    print(f"Błąd HTTP z API: {e.code} {e.reason}", file=sys.stderr)
    sys.exit(1)
except URLError as e:
    print(f"Błąd połączenia z API: {e.reason}", file=sys.stderr)
    sys.exit(1)

if not resp_data.strip():
    print("Błąd: pusta odpowiedź z API.", file=sys.stderr)
    sys.exit(1)

try:
    obj = json.loads(resp_data)
    content = obj["choices"][0]["message"]["content"]
    print(content.strip())
except Exception as e:
    print("Błąd JSON:", e, file=sys.stderr)
    print("Surowa odpowiedź API:", resp_data, file=sys.stderr)
    sys.exit(1)
PYTHON
```
## STEP 5. Create script sgsh on /usr/local/bin/sgsh
```bash
#!/usr/bin/env bash

#Użycie: sgsh "twoje polecenie"
if [ $# -eq 0 ]; then
    echo "Użycie: sgsh \"co chcesz zrobić\"" >&2
    exit 1
fi

QUESTION="$*"

echo
echo "Pytanie do modelu:"
echo "  $QUESTION"
echo

# Pobierz komendę z modelu
RAW_CMD=$(sgpt "Podaj jedną komendę bash, która wykona zadanie: $QUESTION. Zwróć tylko komendę, bez żadnych komentarzy.")

# Wyczyść niewidoczne znaki
CMD=$(echo "$RAW_CMD" | tr -d '\n\r\t' | sed 's/`//g' | sed "s/'//g" | xargs)

echo "Proponowana komenda:"
echo "  $CMD"
echo

read -r -p "Wykonać? [y/N] " ans

case "$ans" in
    y|Y )
        echo
        echo ">>> Wykonuję: $CMD"
        echo "--------------------------------------------"
        eval "$CMD"
        ;;
    * )
        echo "Przerwano."
        ;;
esac
```

## STEP 6. Add file permissions
```bash
chmod +x /usr/local/bin/sgpt
chmod +x /usr/local/bin/sgsh
```

## STEP 7. Using sgpt and sgsh
<img width="783" height="412" alt="image" src="https://github.com/user-attachments/assets/aec6048a-e9a2-41ae-ad26-d19ded4d7ab4" />
<img width="736" height="400" alt="image" src="https://github.com/user-attachments/assets/bea18da1-5d12-4cc7-9e51-d0c196181972" />

<img width="544" height="480" alt="image" src="https://github.com/user-attachments/assets/f05cafe0-e0f8-4c4e-abf4-b057ce0e69d0" />

## STEP 1. We must allow scripts to run. Run powershell as admin
```bash
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

## STEP 2. Create directory on C:/Tools

## STEP 3. Add port 11434/TCP on Windows Defender Firewall with Advanced Security (Inbound and Outbound)

## STEP 4. Create scrip sgpt.ps1 and paste file
```bash
param(
    [Parameter(Mandatory=$true, ValueFromRemainingArguments=$true)]
    [string[]]$PromptParts
)

# Łączymy argumenty w jedno zdanie
$prompt = $PromptParts -join " "

# ADRES TWOJEGO SERWERA LLM
# jeśli serwer jest na innej maszynie, zmień localhost na IP, np. http://192.168.10.50:11434
$apiBase = "http://192.168.10.57:11434"
$uri     = "$apiBase/v1/chat/completions"

# Budujemy JSON dla API
$bodyObject = @{
    model    = "local"
    messages = @(
        @{
            role    = "user"
            content = $prompt
        }
    )
}

$bodyJson = $bodyObject | ConvertTo-Json -Depth 5

try {
    # -ErrorAction Stop => jeśli API zwróci błąd, od razu lecimy do catch
    $resp = Invoke-RestMethod -Uri $uri -Method Post -ContentType "application/json" -Body $bodyJson -ErrorAction Stop
}
catch {
    Write-Error "Błąd połączenia z API: $($_.Exception.Message)"
    return
}

# Sprawdzamy, czy odpowiedź ma pole 'choices'
if (-not $resp -or -not $resp.choices) {
    Write-Error ("Błędna odpowiedź API (brak pola 'choices'). Odpowiedź: " + ($resp | ConvertTo-Json -Depth 5))
    return
}

# Wyciągamy treść odpowiedzi
$text = $resp.choices[0].message.content

if (-not $text) {
    Write-Error "API nie zwróciło treści odpowiedzi."
    return
}

$text.Trim()
```

## STEP 5. Run SGPT on Windows
<img width="993" height="322" alt="image" src="https://github.com/user-attachments/assets/aadfc343-0fe8-4171-916e-6905a9dc7d8b" />


