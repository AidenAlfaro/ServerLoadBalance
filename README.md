# ServerLoadBalance
Ejemplo funcional de proyecto final Server Load Balancing

Paso 1: Instalar Dependencias
1.	Instalar Homebrew (si no está ya instalado):
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
Instala el gestor de paquetes Homebrew.
2.	Instalar HAProxy usando Homebrew:
brew install haproxy
Instala HAProxy.
Paso 2: Limpiar Instancias de HAProxy en Ejecución
1.	Detener todas las instancias de HAProxy en ejecución:
sudo killall haproxy
Detiene todas las instancias de HAProxy para evitar conflictos.
Paso 3: Crear y Verificar Servidores Backend
1.	Crear directorios para los servidores backend:
mkdir ~/backend1 ~/backend2 ~/backend3
Crea directorios para los servidores backend.
2.	Crear un archivo HTML simple en cada directorio backend:
Para backend1:
echo "<html><body><h1>Servidor Backend 1</h1></body></html>" > ~/backend1/index.html
Crea un archivo HTML para el servidor backend 1.
Para backend2:
echo "<html><body><h1>Servidor Backend 2</h1></body></html>" > ~/backend2/index.html
Crea un archivo HTML para el servidor backend 2.
Para backend3:
echo "<html><body><h1>Servidor Backend 3</h1></body></html>" > ~/backend3/index.html
Crea un archivo HTML para el servidor backend 3.
3.	Iniciar los servidores backend:
Terminal 1:
cd ~/backend1
python3 -m http.server 8081 --bind 127.0.0.1
Inicia un servidor HTTP en el puerto 8081 para el backend 1.
Terminal 2:
cd ~/backend2
python3 -m http.server 8082 --bind 127.0.0.1
Inicia un servidor HTTP en el puerto 8082 para el backend 2.
Terminal 3:
cd ~/backend3
python3 -m http.server 8083 --bind 127.0.0.1
Inicia un servidor HTTP en el puerto 8083 para el backend 3.
4.	Verificar acceso a cada servidor backend:
curl http://127.0.0.1:8081
curl http://127.0.0.1:8082
curl http://127.0.0.1:8083
Verifica la conectividad a cada servidor backend para asegurarse de que están funcionando.
Paso 4: Configurar HAProxy
1.	Crear el directorio de configuración de HAProxy:
sudo mkdir -p /usr/local/etc
Crea el directorio para los archivos de configuración de HAProxy.
2.	Crear el archivo de configuración de HAProxy:
sudo nano /usr/local/etc/haproxy.cfg
3.	Agregar la siguiente configuración:
global
    log 127.0.0.1 local0 debug
    log 127.0.0.1 local1 debug
    chroot /usr/local/var/haproxy
    stats socket /usr/local/var/run/haproxy.sock mode 600 level admin
    stats timeout 30s
    user root
    group wheel
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    option logasap
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /
    log global
    server server1 127.0.0.1:8081 check inter 2000 rise 2 fall 3
    server server2 127.0.0.1:8082 check inter 2000 rise 2 fall 3
    server server3 127.0.0.1:8083 check inter 2000 rise 2 fall 3
Configura HAProxy para registrar información de depuración, definir configuraciones de frontend y backend, y especificar verificaciones de salud para los servidores backend.
4.	Guardar el archivo y salir del editor (presiona CTRL + X, luego Y y Enter).
Paso 5: Iniciar HAProxy en Modo Debug
1.	Iniciar HAProxy con la nueva configuración:
sudo haproxy -d -f /usr/local/etc/haproxy.cfg
Inicia HAProxy en modo depuración para ver información detallada en el terminal.
Paso 6: Verificar Conectividad de HAProxy
1.	Abrir tu navegador y navegar a la página de estadísticas de HAProxy:
http://localhost/haproxy?stats
Accede a la página de estadísticas de HAProxy para monitorear el estado de tus servidores backend.
Paso 7: Comprobar Errores en la Configuración de HAProxy
1.	Ejecutar el siguiente comando para comprobar la configuración de HAProxy:
sudo haproxy -c -f /usr/local/etc/haproxy.cfg
Verifica la configuración de HAProxy en busca de errores de sintaxis.
Paso 8: Asegurar un Registro Apropiado
1.	Asegurarse de que los registros de HAProxy se escriban en un archivo para una mejor diagnóstico:
Editar la configuración de rsyslog:
sudo nano /etc/rsyslog.conf
2.	Agregar la siguiente línea para asegurarse de que se capturen los registros de HAProxy:
if ($programname == 'haproxy') then /var/log/haproxy.log
& stop
3.	Reiniciar el servicio rsyslog:
sudo systemctl restart rsyslog
4.	Verificar los registros de HAProxy:
sudo tail -f /var/log/haproxy.log
Estos comandos configuran rsyslog para capturar los registros de HAProxy y reinician el servicio de registro para aplicar los cambios. El último comando te permite ver los registros de HAProxy en tiempo real.
Resumen de Comandos y Pasos
1.	Instalar dependencias:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install haproxy
2.	Detener todas las instancias de HAProxy en ejecución:
sudo killall haproxy
3.	Crear y verificar servidores backend:
Crear directorios:
mkdir ~/backend1 ~/backend2 ~/backend3
Crear archivos HTML:
echo "<html><body><h1>Servidor Backend 1</h1></body></html>" > ~/backend1/index.html
echo "<html><body><h1>Servidor Backend 2</h1></body></html>" > ~/backend2/index.html
echo "<html><body><h1>Servidor Backend 3</h1></body></html>" > ~/backend3/index.html
Iniciar servidores backend:
cd ~/backend1
python3 -m http.server 8081 --bind 127.0.0.1

cd ~/backend2
python3 -m http.server 8082 --bind 127.0.0.1

cd ~/backend3
python3 -m http.server 8083 --bind 127.0.0.1
Verificar acceso:
curl http://127.0.0.1:8081
curl http://127.0.0.1:8082
curl http://127.0.0.1:8083
4.	Crear directorio de configuración de HAProxy:
sudo mkdir -p /usr/local/etc
5.	Crear y editar archivo de configuración de HAProxy:
sudo nano /usr/local/etc/haproxy.cfg
6.	Agregar configuración de HAProxy:
global
    log 127.0.0.1 local0 debug
    log 127.0.0.1 local1 debug
    chroot /usr/local/var/haproxy
    stats socket /usr/local/var/run/haproxy.sock mode 600 level admin
    stats timeout 30s
    user root
    group wheel
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    option logasap
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /
    log global
    server server1 127.0.0.1:8081 check inter 2000 rise 2 fall 3
    server server2 127.0.0.1:8082 check inter 2000 rise 2 fall 3
    server server3 127.0.0.1:8083 check inter 2000 rise 2 fall 3
7.	Iniciar HAProxy en modo debug:
sudo haproxy -d -f /usr/local/etc/haproxy.cfg
8.	Verificar registros de HAProxy:
log stream --predicate 'process == "haproxy"' --info
9.	Comprobar página de estadísticas de HAProxy:
http://localhost/haproxy?stats
10.	Comprobar configuración de HAProxy en busca de errores de sintaxis:
sudo haproxy -c -f /usr/local/etc/haproxy.cfg
11.	Asegurar un registro apropiado:
sudo nano /etc/rsyslog.conf
if ($programname == 'haproxy') then /var/log/haproxy.log
& stop
sudo systemctl restart rsyslog
sudo tail -f /var/log/haproxy.log

![image](https://github.com/user-attachments/assets/1ea145c0-1912-491a-b1cc-f46dd437daf0)
