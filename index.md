

# Proyecto Redes Grupo 2

[Link a respositorio de GitHub](https://github.com/Criplockoweno/Dropbox_Proyect_Redes).

#### Integrantes:
- Esteban Alvarado
- Cristopher Becerra
- José Contreras
- Kristian Mendoza
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

Un punto a destacar es que en el proyecto utilizamos en su mayoría código hecho por nosotros mismos, no utilizamos librerias diseñadas especificamente para hacer un handle de HTML. Utilizamos las librerías de  sockets, threading, base64, bs4. 

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
    request = b''
    while True:
        request = conn.recv(HEADER)
        if not request:
            break
        httpd = HTTPRequestHandler(request, conn)
        response = httpd.handle_request()
        conn.sendall(response.encode(FORMAT))
       
    conn.close()

def start():
    
    server.listen()
    print(f"[LISTENING] Server is listening on {SERVER}")
    while True:
        conn, addr = server.accept()
        thread = threading.Thread(target = handle_client, args = (conn, addr))
        thread.start()
        print(f"[ACTIVE CONNECTIONS]= {threading.active_count()-1}")


if __name__ == "__main__":
    server=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(ADDR)
    print("Server is starting...")
    start()

```
Se realizó un servidor web básico que escucha las solicitudes HTTP en un puerto específico. La función principal del servidor es crear un socket y esperar las solicitudes entrantes. Cuando se recibe una solicitud, se crea un nuevo hilo para manejar la solicitud y se envía una respuesta al cliente. El servidor se ejecuta en un bucle infinito.

# HTTPRequestHandler

La clase se encarga de manejar las solicitudes HTTP que llegan al servidor.

- El método 'handle_chunked_request()' se encarga de procesar las solicitudes en formato chunked. El metodo recibe los fragmentos de la solicitud en bloques de tamaño fijo y los procesa hasta que se ha recibido todo el contenido de la solicitud.
```py
 def handle_chunked_request(self, request, content_length):
    while content_length > 0:
        chunk = self.conn.recv(4096)
        content_length -= len(chunk)
        request += chunk
    return request

```

- El método 'handle_file_request()'  maneja las solicitudes de carga de archivos. Se extrae el nombre del archivo, el tipo de contenido, y los datos relacionados. Se devuelve una tupla de los anteriores campos mencionados.
```py
def handle_file_request(self, request, headers):
    boundaries = '--' + \
        re.search(r'boundary=([^;\\]+)', headers['Content-Type']).group(1)
    request_body = request.split(b'\r\n\r\n', 1)[1]
    start = request_body.find(boundaries.encode('utf-8')) + len(boundaries)
    end = request_body.find(boundaries.encode('utf-8'), start)
    bh, filedata = request_body[start:end].split(b'\r\n\r\n', 1)
    bh = bh.decode('utf-8').split('\r\n')
    body_headers = {}
    for line in bh:
        if line.strip() != '':
            key, value = line.split(': ')
            body_headers[key] = value.strip()
    filename = re.search(
        r'filename=([^;\\]+)', body_headers['Content-Disposition']).group(1).replace('"', '')
    content_type = body_headers['Content-Type']
    return filename, filedata, content_type
```

- El método 'parse_request()' analiza los encabezados de la solicitud y su contenido, y lo devuelve como un diccionario. Si la solicitud contiene datos en su cuerpo, se llama al método 'handle_chunked_request()' para manejar dichos datos. 
```py
 def parse_request(self, request):
    headers = {}
    lines = request.decode('utf-8').split('\r\n')
    method, path, protocol = lines[0].split(" ")
    headers.update({'Method': method, 'Path': path, 'Protocol': protocol})
    for line in lines[1:]:
        if line.strip() != '':
            key, value = line.split(': ')
            headers[key] = value.strip()
    if 'Content-Length' in headers:
        request = self.handle_chunked_request(
            request, int(headers['Content-Length']))
    return headers, request
```

- El método 'handle_request()' se manejan los diferentes tipos de request que pueden recibirse en el el servidor. Se comparan los Metodos que están guardados en el header con un GET y POST para decidir qué se debe hacer y que se responde al cliente.
``` py
def handle_request(self):
    headers, request = self.parse_request(self.request)
    response = ""

    if headers['Method'] == 'GET':
        response = self.do_GET(headers)

    if headers['Method'] == 'POST':
        response = self.do_POST(headers, request)

    return response


```

- El método 'do_GET()' se encarga de manejar las solicitudes GET. Esta función comprueba si la ruta solicitada en la solicitud es la raíz del servidor (ruta "/"). Si es así, abre el archivo "index.html" y lee su contenido con el objeto open y lo almacena en la variable html. Si no es así, se abre el archivo solicitado y se lee su contenido. El contenido del archivo se envía al cliente como una respuesta HTTP. Luego crea una respuesta HTTP con un código 200, establece el tipo de contenido y la longitud. 
```py
def do_GET(self, headers):
    if headers['Path'] == '/':
        with open("index.html", "r", encoding='utf-8') as f:
            html = HTMLPreprocessing(f.read()).get_processed_html()
        response = ('HTTP/1.1 200 OK\r\n'
                    + 'Content-Type: text/html\r\n'
                    + 'Content-Length: {}\r\n'.format(len(html))
                    + '\r\n' + html)
        return response
```

- El método 'do_POST()' maneja los request POST del cliente. Se lee en la lista de encabezados si es que el path es '/upload' para utilizar el método handle_file_request(). Luego se utiliza una instancia del FileManager y un método de esta clase para guardar los datos es el directorio './files'. Luego se retorna una respuesta al cliente que permite actualizar la página.
```py
def do_POST(self, headers, request):
    if headers['Path'] == '/upload':
        filename, filedata, content_type = self.handle_file_request(
            request, headers)
        file_manager = FileManager('./files/')
        file_manager.save_file_on_directory(
            filename, filedata, content_type)
        response = "HTTP/1.1 301 Moved Permanently\r\nLocation: / \r\n\r\n"
        return response
```

# HTMLPreprocessing
``` py
class HTMLPreprocessing:
    
    def __init__(self, html):
        self.html = html
        
    def get_processed_html(self):
        return self.process_html()

    def process_html(self):
        soup = BeautifulSoup(self.html, 'html.parser')
        file_manager = FileManager('./files')
        for filename in file_manager.get_files_on_directory():
            path, size, date = file_manager.get_file_data(filename)
            encoded_content = file_manager.encode_file_content(path)
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
Esta clase se encarga de procesar los archivos HTML. El método 'process_html()' utiliza la biblioteca BeautifulSoup para anlizar el HTML y encontrar las etiquetas 'div' con el atributo 'id' igual a 'filesList'. Luego utiliza una instancia de la clase FileManager para obtener la información de los archivos en un directorio y generar una lista de archivos HTML. El método 'get_file_html()' devuelve un codigo HTML que es un contenedor de tarjeta con una imagen de archivo y dos secciones, una con el nombre del archivo y un botón de descarga. Adicionalmente se presenta información adicional como el tamaño del archivo y la fecha de envío.

# FileManager


La clase File manager permite manipular archivos del directorio específico del servidor. Se tienen diferentes métodos, por ejemplo: 

- El método 'get_file_data()' que permite devolver la ruta del archivo, su tamaño en bytes  y su fecha de creación. 
```py
 def get_file_data(self, filename):
    path = os.path.join(self.directory, filename)
    size = self.get_file_size_str(os.path.getsize(path))
    date = time.ctime(os.path.getctime(path))
    return path, size, date
```
- El método 'encode_file_content()' toma la ruta de un archivo como argumento, y lo codifica en Base64, y devuelve el contenido codificado como Unicode.
```py
def encode_file_content(self, file_path):
    with open(file_path, 'rb') as f:
        content = f.read()
        encoded_content = b64encode(content).decode('utf-8')
    return encoded_content
```
- El método de 'get_files_on_directory()' devuelve una lista de los nombre de los archivos en el directorio especificado. 
```py
def get_files_on_directory(self):
    return [filename for filename in os.listdir(self.directory)]

```
- El método 'get_file_size_str' devuelve el tamaño del archivo en bytes, kilobytes o megabytes, dependiendo de su tamaño.
```py
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
```
- El método 'save_file_on_directory()' toma el nombre del archivo,sus datos y su tipo de conteido y guarda el archivo en el directorio. Dependiendo de qué tipo de contenido es, lo guarda en el directorio como archivo binario o como archivo de texto. 
```py
def save_file_on_directory(self, file_name, file_data, file_type):
    if (file_type == ('text/plain' or 'text/html' or 'application/json')):
        with open(os.path.join(self.directory, file_name), 'w', encoding='utf-8') as f:
            f.write(file_data.decode('utf-8'))
    else:
        with open(os.path.join(self.directory, file_name), 'wb') as f:
            f.write(file_data)
```

# Conclusiones

1. El manejo de solicitudes HTTP requiere la manipulación de headers para llevar a cabo una respuesta adecuada. Los headers llevan información importante sobre la solicitud del cliente como: método, protocolo, ruta, tamaño del contenido, etc.   
2. La carga y descarga de archivos requieren de una codificación y decodificación para que estos puedan ser manejados dentro de las solicitudes y respuestas HTTP. Archivos de texto plano se pueden codificar en UTF-8, pero archivos como PDF o imágenes requieren de una codificación binaria para guardarse en el servidor y posteriormente descargarse.
3. El tamaño de las solicitudes del cliente tienen un tamaño limitado (particularmente de 4096 bytes). Si el usuario hace una solicitud que requiere enviar un archivo de gran tamaño será necesario recibir todos los paquetes HTTP que componen todo el contenido del archivo. Esta situación se puede manejar mediante el análisis de los headers HTTP, en especial del Content-Length.
3. La descarga de los archivos se realiza 

