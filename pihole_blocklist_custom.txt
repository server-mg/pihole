#!/bin/bash
# Script automático para atualizar listas no Pi-hole com log

LIST_URL="https://raw.githubusercontent.com/SEU_USUARIO/REPO/main/pihole_blocklist_custom.txt"
LIST_FILE="/tmp/pihole_blocklist_custom.txt"
LOG_FILE="/var/log/pihole_update.log"

echo "[2025-05-26 08:10:56.681073] Iniciando atualização do Pi-hole..." >> "$LOG_FILE"

# Baixar a lista externa
echo "[+] Baixando lista personalizada..." >> "$LOG_FILE"
curl -fsSL "$LIST_URL" -o "$LIST_FILE"
if [ $? -ne 0 ]; then
  echo "[!] Erro ao baixar a lista personalizada de $LIST_URL" >> "$LOG_FILE"
  exit 1
fi

# Importar domínios manuais
echo "[+] Importando domínios manuais..." >> "$LOG_FILE"
grep -v '^#' "$LIST_FILE" | grep -v 'http' | grep -Ev '^\s*$' | while read -r domain; do
  if [[ ! "$domain" =~ (\^|\\\.) ]]; then
    pihole -b "$domain" >> "$LOG_FILE" 2>&1
  fi
done

# Importar regex personalizados
echo "[+] Importando regex..." >> "$LOG_FILE"
grep -E '(^\^|\\\.)' "$LIST_FILE" | while read -r regex; do
  pihole --regex "$regex" >> "$LOG_FILE" 2>&1
done

# Importar listas externas
echo "[+] Adicionando listas externas..." >> "$LOG_FILE"
grep -E '^https?://' "$LIST_FILE" | while read -r url; do
  pihole -g -addlist "$url" >> "$LOG_FILE" 2>&1
done

# Atualizar Gravity
echo "[+] Atualizando gravity..." >> "$LOG_FILE"
pihole -g >> "$LOG_FILE" 2>&1

echo "[✔] Finalizado em $(date)" >> "$LOG_FILE"
