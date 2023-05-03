---
layout: default
---

[Link a respositorio de GitHub](./another-page.html).

# Proyecto Redes Grupo 2
- Cristopher Becerra
- Kristian Mendoza
- José Contreras
- Esteban Alvarado
- José Luis Santillán

# Objetivos

- Implementar una interfaz de usuario web en la que usuarios carguen y descarguen
archivos.
- Crear un servidor web en Python utilizando sockets para responder las solicitudes
del cliente/consumidor.
- Implementar el protocolo http para permitir la comunicación entre la aplicación
cliente y el servidor.

# Descripción

La implementación de una interfaz de usuario web es uno de los principales componentes de este proyecto. Esta interfaz permitirá a los usuarios cargar y descargar archivos de manera eficiente y fácil. Los usuarios podrán navegar a través de la interfaz para buscar los archivos que necesitan, y podrán subir sus propios archivos al servidor.

Para permitir la comunicación entre el servidor y los clientes, se utilizará el protocolo HTTP. Este es el protocolo estándar utilizado para la comunicación entre aplicaciones web y servidores. Se asegurará de que la aplicación se ajuste a los estándares HTTP y de que esté optimizada para proporcionar una experiencia de usuario rápida y fluida.

El servidor será implementado en Python utilizando sockets de red. Un socket es una abstracción de software que permite la comunicación entre dos procesos en una red. Los sockets de red son la base de la comunicación en Internet y se utilizan para enviar y recibir datos entre los clientes y los servidores. 

La implementación de un servidor web en Python utilizando sockets de red permitirá a la aplicación responder a las solicitudes de los clientes de manera rápida y eficiente. Esto garantizará que los usuarios puedan cargar y descargar archivos de manera rápida y sin interrupciones, lo que proporcionará una experiencia de usuario más satisfactoria.


# Servidor

```py
import socket
import threading
from HTTPRequestHandler import HTTPRequestHandler

HEADER=4096
PORT=5053
SERVER=socket.gethostbyname(socket.gethostname())
ADDR=(SERVER,PORT)
FORMAT='utf-8'
DISCONNECT_MESSAGE="!DISCONNECT"

def handle_client(conn, addr):
    
    print(f"[NEW CONNECTION]{addr} connected.")
    while True:
        # recibimos un mensaje de longitud maxima de 64 bytes
        # y lo decodificamos en formato UTF-8
        request = conn.recv(HEADER).decode(FORMAT)
        print(request)
        if not request:
            break
        httpd = HTTPRequestHandler(request)
        response = httpd.handle_request()
        # Enviamos la respuesta HTTP al cliente
        conn.sendall(response.encode(FORMAT))
       
    conn.close()

def start():
    
    server.listen()
    print(f"[LISTENING] Server is listening on {SERVER}")
    while True:
        # esperamos una conexion y cuando llegue la aceptamos
        conn, addr = server.accept()
        thread = threading.Thread(target = handle_client, args = (conn, addr))
        thread.start()
        print(f"[ACTIVE CONNECTIONS]= {threading.active_count()-1}")


if __name__ == "__main__":
    # creamos la instancia Socket
    server=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # asignamos la direccion IP y el puerto al socket
    server.bind(ADDR)
    print("Server is starting...")
    start()

```

# HTTPRequestHandler
``` py

from HTMLPreprocessing import HTMLPreprocessing

class HTTPRequestHandler():
    
    def __init__(self, request):
        self.request = request
        self.FORMAT = 'utf-8'
    
    def parse_request(self, request):
        headers = {}
        lines = request.split('\n')

        if(lines[0].find('----WebKitFormBoundary')!=-1):
            # Split the data by the boundary string
            parts = request.split('------WebKitFormBoundary')

            # Find the part that contains the file data
            for part in parts:
                if 'Content-Disposition: form-data; name="myFile"' in part:
                    # Extract the filename and file contents
                    filename = part.split('filename="')[1].split('"')[0]
                    filedata = part.split('\r\n\r\n')[1].split('\r\n------')[0]
                    print(filedata)
            headers.update({'Method': 'Webkit', 'Path': '/', 'Protocol': 'HTTP/1.1', 'Content-Disposition': 'form-data', 'filename': filename, 'filedata': filedata})
        else:
            method, path, protocol = lines[0].split(" ")
            headers.update({'Method': method, 'Path': path, 'Protocol': protocol})
            for line in lines[1:]:
                if line.strip() != '':
                    key, value = line.split(': ')
                    headers[key] = value.strip()
        

        return headers

    
    def handle_request(self):
        headers = self.parse_request(self.request)
        response = ""
        if headers['Method'] == 'GET':
            response = self.do_GET()
        
        if headers['Method'] == 'POST':
            response = self.do_POST()

        if headers['Method'] == 'Webkit':
            response = self.writeData()
        return response
        
    def do_GET(self):
        with open("index.html", "r", encoding=self.FORMAT) as f:
            html = HTMLPreprocessing(f.read()).get_processed_html()
        response = ('HTTP/1.1 200 OK\r\n' 
            + 'Content-Type: text/html\r\n' 
            + 'Content-Length: {}\r\n'.format(len(html)) 
            + '\r\n' + html)
        return response
    
    def do_POST(self):
        response = "HTTP/1.1 303 See Other\r\nLocation: /\r\n\r\n"
        return response
    
    def writeData(self):
        headers = self.parse_request(self.request)
        with open('./files/' + headers['filename'], "wb") as f:
            f.write(headers['filedata'].encode(self.FORMAT))        
        response = "HTTP/1.1 200 OK\r\n\r\nFile writed successfully"
        return response
    


```



# HTMLPreprocessing
``` py
from bs4 import BeautifulSoup
from FileManager import FileManager
import base64
class HTMLPreprocessing:
    
    def __init__(self, html):
        self.html = html
        
    def get_processed_html(self):
        return self.process_html()
    
    def encode_file_content(self, file_path):
        with open(file_path, 'rb') as f:
            content = f.read()
            encoded_content = base64.b64encode(content).decode('utf-8')
        return encoded_content

    def process_html(self):
        soup = BeautifulSoup(self.html, 'html.parser')
        file_manager = FileManager('./files')
        for filename in file_manager.get_files_on_directory():
            path, size, date = file_manager.get_file_data(filename)
            encoded_content = self.encode_file_content(path)
            list_file_html = self.get_file_html(filename, size, encoded_content, date)
            file_list_container = soup.find('div', {'id': 'filesList'})                
            file_list_container.append(BeautifulSoup(list_file_html, 'html.parser'))
        return soup.prettify(formatter="html")
    
    def get_file_html(self, file_name:str, size:str, encoded_content:str, submission_date:str):
        return f"""<div class="card my-2">
                    <div class="card-body d-flex justify-content-between align-items-center">
                        <p class="card-text my-0">{file_name}</p>
                        <a class="my-0" href="data:application/octet-stream;base64,{encoded_content}" download="{file_name}">
                            <svg xmlns="http://www.w3.org/2000/svg" width="25" height="25" fill="currentColor" class="bi bi-file-arrow-down-fill" viewBox="0 0 16 16">
                                <path d="M12 0H4a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2V2a2 2 0 0 0-2-2zM8 5a.5.5 0 0 1 .5.5v3.793l1.146-1.147a.5.5 0 0 1 .708.708l-2 2a.5.5 0 0 1-.708 0l-2-2a.5.5 0 1 1 .708-.708L7.5 9.293V5.5A.5.5 0 0 1 8 5z"/>
                            </svg>
                        </a>
                    </div>
                    <div class="card-footer d-flex justify-content-between text-body-secondary">
                        <p class="mb-0">{size}</p>
                        <p class="mb-0">Submitted on {submission_date}</p>
                    </div>
                </div>"""
```
# FileManager

```py
import os
import time

class FileManager():
    def __init__(self, directory):
        self.directory = directory
    
    def get_file_data(self, filename):
        path = os.path.join(self.directory, filename)
        size = self.get_file_size_str(os.path.getsize(path))
        date = time.ctime(os.path.getctime(path))
        return path, size, date
    
    def get_files_on_directory(self):
        return [filename for filename in os.listdir(self.directory)]
    
    def get_file_size_str(self, size_bytes):
        units = ('B', 'KB', 'MB', 'GB')
        size_thresholds = (1, 1024, 1024**2, 1024**3)

        for i, threshold in enumerate(size_thresholds):
            if size_bytes < threshold:
                size = size_bytes / size_thresholds[i-1]
                unit = units[i-1]
                break
        else:
            size = size_bytes / size_thresholds[-1]
            unit = units[-1]
            
        size_str = f"{size:.1f} {unit}"
        return size_str
    
    def save_file_on_directory(self):
        # Maybe here we can control the save of a file
        print('')
```

# Conclusiones

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.


### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)

