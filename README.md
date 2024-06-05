import requests


# Аутентификация в ThingsBoard
def authenticate(url, username, password):
    auth_url = f"{url}/api/auth/login"
    payload = {
        "username": username,
        "password": password
    }
    response = requests.post(auth_url, json=payload)
    if response.status_code == 200:
        return response.json()['token']
    else:
        print("Authentication failed:", response.status_code, response.json())
        return None


# Сбор телеметрии с ThingsBoard PE
def get_telemetry(token, url, device_id):
    telemetry_url = f"{url}/api/plugins/telemetry/DEVICE/{device_id}/values/timeseries"
    headers = {
        'accept': 'application/json',
        'X-Authorization': f'Bearer {token}'
    }
    response = requests.get(telemetry_url, headers=headers)
    if response.status_code == 200:
        return response.json()
    else:
        print("Failed to get telemetry:", response.status_code, response.json())
        return None


# Отправка телеметрии на ThingsBoard CE
def send_telemetry(token, url, device_id, telemetry_data):
    telemetry_url = f"{url}/api/v1/{device_id}/telemetry"
    headers = {
        'Content-Type': 'application/json',
        'X-Authorization': f'Bearer {token}'
    }
    response = requests.post(telemetry_url, json=telemetry_data, headers=headers)
    if response.status_code == 200:
        print("Telemetry sent successfully")
    else:
        print("Failed to send telemetry:", response.status_code, response.json())


# Основная программа
tb_pe_url = "http://localhost:8080"  # URL ThingsBoard PE
tb_ce_url = "http://10.7.2.159:8080"  # URL ThingsBoard CE
username = "tenant@thingsboard.org"
password = "tenant"

# Получение токенов аутентификации
pe_token = authenticate(tb_pe_url, username, password)
ce_token = authenticate(tb_ce_url, username, password)

if pe_token and ce_token:
    print(f"PE Token: {pe_token}")
    print(f"CE Token: {ce_token}")

    # Получение телеметрии с устройства в ThingsBoard PE
    pe_device_id = "69c0d3e0-1f34-11ef-adea-f366d2a9ddce"
    telemetry_data = get_telemetry(pe_token, tb_pe_url, pe_device_id)

    if telemetry_data:
        print(f"Telemetry Data: {telemetry_data}")
        # Отправка телеметрии на устройство в ThingsBoard CE
        ce_device_id = "d6367250-1f4d-11ef-8992-094cb3eb5a17"  # Обратите внимание на ID устройства в CE
        send_telemetry(ce_token, tb_ce_url, ce_device_id, telemetry_data)
else:
    print("Failed to authenticate in one or both instances of ThingsBoard.")
