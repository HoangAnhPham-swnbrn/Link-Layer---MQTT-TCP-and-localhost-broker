## TWO DEVICES MQTT PING AND CHAT - INSTRUCTIONS

| --- | Device A | Device B | 
| --- | --- | --- |
| Terminal | VS Code (Powershell) | Command Prompt |
| Runs Mosquitto broker | Yes | No |
| Has '''mosquitto_lan.conf''' | Yes | No |
| Has '''mqtt_combined.py''' | Yes | Yes |
| Role argument | '''a''' | '''b''' |

### Before you start (on both devices)
- Both devices are on the same local network so B can find A's directly (Eduroam usually isolates devices even they share the same Wi-Fi name).
- VPN is off on device A.
- Each laptop has its own copy of '''mqtt_combined.py'''.

### Device A: Terminal 1 (VS Code, Powershell)
1. Go to the project folder and check the config has both lines ('''listener 1884 0.0.0.0''' and '''allow_anonymous true'''):

'''@"
listener 1884 0.0.0.0
allow_anonymous true
"@ | Set-Content -Encoding ascii mosquitto_lan.conf'''

2. Confirm your Wi-Fi IPv4:

'''(Get-NetIPAddress -InterfaceAlias Wi-Fi -AddressFamily IPv4).IPAddress'''

3. Stop any broker if accidentally starter before:

'''Get-CimInstance Win32_Process -Filter "Name='mosquitto.exe'" | Where-Object CommandLine -like '*mosquitto_lan.conf*' | ForEach-Object { Stop-Process -Id $_.ProcessId }'''

4. Start the broker in the background:

'''Start-Process "C:\Program Files\mosquitto\mosquitto.exe" -ArgumentList '-c','.\mosquitto_lan.conf','-v' -WindowStyle Hidden -RedirectStandardError broker.log'''

5. Checking it is listening (expecting '''0.0.0.0:1884''' and '''LISTENING'''):

'''netstat -an | findstr 1884'''

### Device A: Terminal 2 (optional, broker log)
- Click the '''+''' in the VS Code terminal panel, then run:

'''Get-Content .\broker.log -Wait'''

- Leave it running. Expect to see '''New connection from ...''' and '''Received PUBLISH ...''' lines.

### Device B (Command Prompt)
1. Open Command Prompt in the folder with the file (File Explorer address bar, type '''cmd''', press Enter).

2. Check B can reach the broker. Confirm '''TcpTestSucceeded : True''':

'''powershell -c "Test-NetConnection <A's IPv4> -Port 1884"'''

### Start the chat
1. Device A: 

'''py mqtt_combined.py a'''

2. Device B:

'''py mqtt_combined.py b --broker <A's IPv4>'''

3. Wait for '''Subscription confirmed''' on both.

### Test
1. Greet A from B, B shows '''SENT - Broker acknowledged''' and A shows a green '''B: <text>'''.

2. Reply from A. It appears on B as a green '''A: <text>'''.

### Finish
1. Type '''quit''' in both chat windows.

2. In the log terminal, press Ctrl+C.

3. Stop the broker on A (Ctrl+C).

### Troubleshooting
| Problem | Check |
| --- | --- |
| '''py''' is not recognized on B | Install Python: '''winget install -e --id Python.Python.3.13''', then open a new Command Prompt |
| '''unrecognized arguments''' | Type '''--broker''' with two dashes and no spaces, all in single line |
| '''No such file or directory''' | Open Command Prompt inside the file's folder or run '''dir *.py''' to check the name, or alternatively, click and drag the the .py file straight into Command Prompt |
| Connection timed out on B | A's IP may have changed; re-run A's by repeating the instructions above | 
| Messages do not arrive | A must use role '''a''', B must use '''b'''; both must show '''Subscription confirmed''' |
| Need broker details | On A, type '''quit''', then run '''Get-Content .\broker.log -Tail 20''' |
