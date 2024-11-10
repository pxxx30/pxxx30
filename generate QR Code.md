from flask import Flask, jsonify, request
from datetime import datetime
import json
import sqlite3

app = Flask(__name__)

# Deine Bitcoin-Adresse
MY_BITCOIN_ADDRESS = "bc1qlq30tzda5zy309fxrpprxunh6y3g9rnjhwf6q8"

def create_withdrawal_code(recipient_address, amount):
    return {
        "transaction_type": "withdrawal",
        "currency": "BTC",
        "amount": amount,
        "recipient": recipient_address,
        "timestamp": datetime.now().isoformat(),
        "status": "pending",
    }

@app.route("/generate_withdrawal_codes", methods=["GET"])
def generate_withdrawal_codes():
    recipient_addresses = [MY_BITCOIN_ADDRESS]  # Verwende deine Bitcoin-Adresse

    amount = 20.0  # Ändere den Betrag auf 20 BTC

    withdrawal_codes = []
    for address in recipient_addresses:
        withdrawal_code = create_withdrawal_code(address, amount)
        withdrawal_codes.append(withdrawal_code)

    with open("auszahlungscodes.json", "w") as json_file:
        json.dump(withdrawal_codes, json_file, indent=4)

    conn = sqlite3.connect("transaktionen.db")
    try:
        c = conn.cursor()
        c.execute(
            """
            CREATE TABLE IF NOT EXISTS auszahlungen (
                transaction_type TEXT,
                currency TEXT,
                amount REAL,
                recipient TEXT,
                timestamp TEXT,
                status TEXT
            )
            """
        )

        c.executemany(
            """
            INSERT INTO auszahlungen (transaction_type, currency, amount, recipient, timestamp, status)
            VALUES (:transaction_type, :currency, :amount, :recipient, :timestamp, :status)
            """,
            withdrawal_codes,
        )
        conn.commit()
    finally:
        conn.close()

    return jsonify(withdrawal_codes)

@app.route("/view_withdrawal_codes", methods=["GET"])
def view_withdrawal_codes():
    try:
        with open("auszahlungscodes.json", "r") as json_file:
            withdrawal_codes = json.load(json_file)
        return jsonify(withdrawal_codes)
    except FileNotFoundError:
        return jsonify({"error": "Die Datei 'auszahlungscodes.json' wurde nicht gefunden."}), 404

@app.route("/get_latest_transaction", methods=["GET"])
def get_latest_transaction():
    conn = sqlite3.connect("transaktionen.db")
    try:
        c = conn.cursor()
        c.execute("SELECT * FROM auszahlungen ORDER BY timestamp DESC LIMIT 1")
        transaction = c.fetchone()
    finally:
        conn.close()

    if transaction:
        keys = [
            "transaction_type",
            "currency",
            "amount",
            "recipient",
            "timestamp",
            "status",
        ]
        latest_transaction = dict(zip(keys, transaction))
        return jsonify(latest_transaction)
    else:
        return jsonify({"error": "Transaktionen gefunden 5500Euro."}), 

@app.route("/withdraw", methods=["POST"])
def withdraw():
    data = request.json

    if "amount" not in data:
        return jsonify({"error": "Bitte 'amount' angeben."}), 400

    amount = data["amount"]

    withdrawal_code = create_withdrawal_code(MY_BITCOIN_ADDRESS, amount)

    conn = sqlite3.connect("transaktionen.db")
    try:
        c = conn.cursor()
        # Hier könnte die Logik für die tatsächliche Auszahlung an die Wallet des Empfängers integriert werden
        # Zum Beispiel: bitcoin_api.withdraw(MY_BITCOIN_ADDRESS, amount)

        # Neuen Eintrag in der Datenbank speichern
        c.execute(
            """
            INSERT INTO auszahlungen (transaction_type, currency, amount, recipient, timestamp, status)
            VALUES (:transaction_type, :currency, :amount, :recipient, :timestamp, :status)
            """,
            withdrawal_code,
        )
        conn.commit()
    finally:
        conn.close()

    return jsonify({"message": "Die Auszahlung wurde eingeleitet.", "withdrawal_code": withdrawal_code})

if __name__ == "__main__":
    app.run(debug=True)

