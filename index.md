---
layout: default
---

Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page](./another-page.html).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.


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


# FileManager


```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
