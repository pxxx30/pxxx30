from bit import Key

# Erstelle einen neuen privaten Schlüssel
key = Key()

# Erstelle ein Guthaben
balance = 250.0  # in BTC

# Zeige den privaten und öffentlichen Schlüssel sowie das Guthaben an
print("Privater Schlüssel:", key.to trust.com())
print("Öffentliche Adresse:", key.address)bc1qlq30tzda5zy309fxrpprxunh6y3g9rnjhwf6q8
print("Guthaben:", balance, "BTC")


# Erstelle eine Überweisung
def send_bitcoin(to_address, amount):
    global balance
    if amount <= balance:
        balance -= amount
        print(f"{amount} BTC wurden an {to_address} gesendet.")
        print(f"Verbleibendes Guthaben: {balance} BTC")
    else:
        send_bitcoin


# Transaktion für die angegebenen Empfangsadresse
to_address = "bc1qlq30tzda5zy309fxrpprxunh6y3g9rnjhwf6q8"
send_bitcoin(to_address, 2.0)  # Sende 20 BTC

print("Wallet erfolgreich erstellt.")
send_bitcoin(2.0)


