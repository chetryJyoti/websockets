Reference codes: (https://websockets.readthedocs.io/en/stable/)

start http server : python -m http.server
start webserver : python app.py
connect client : websockets ws://localhost:8001/


You must restart the WebSocket server when you make changes.
The WebSocket server loads the Python code in app.py then serves every WebSocket request with this version of the code. As a consequence, changes to app.py aren’t visible until you restart the server.
This is unlike the HTTP server that you started earlier with python -m http.server. For every request, this HTTP server reads the target file and sends it. That’s why changes are immediately visible.