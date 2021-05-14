### TP3 Sistemas Operativos II
## Ingeniería en Compuatación - FCEFyN - UNC
# Sistemas Embebidos

## Introducción
Los _sistemas embebidos_ suelen ser accedidos de manera remota, una forma común, suelen ser las _RESTful APIs_. Estas, brindan una interfaz definida y robusta para la comunicación y manipulación del _sistema embebido_ de manera remota. Definidas para un esquema _Cliente-Servidor_ se utilizan en todas las verticales de la industria tecnológica, desde aplicaciones de _IoT_ hasta juegos multijugador.

## Objetivo
El objetivo del presente trabajo practico es que el estudiante tenga un visión \textit{end to end} de una implementación básica de una \textit{RESTful API} sobre un \textit{sistema enbedido}.
El estudiante deberá implementarlo interactuando con todas las capas del procesos. Desde el \textit{testing} funcional (alto nivel) hasta el código en C del servicio (bajo nivel).

## Desarrollo
### Requerimientos
Para realizar el presente trabajo practico, es necesario una computadora con _kernel_ GNU/Linux, ya que usaremos _SystemD_ [ref] para implementar el manejo de nuestro servicios.

### Desarrollo
Se deberá implementar dos servicios en lenguaje C, estos son el _servicio de usuarios_ y el _servicio de status_. Ambos servicios, deberán exponer una _REST API_. Con el objetivo de acelerar el proceso de desarrollo vamos a utilizar un _framework_: se propone utilizar https://github.com/babelouest/ulfius. El estudiante puede seleccionar otro, justificando la selección, o implementar el propio (no recomendado).
El servicio debe tener configurado un _nginx_ [ref] por delante para poder direccionar el _request_ al servicio correspondiente.
El web server, deberá autenticar el _request_ por medio de de un usuario y password enviado en el _request_, definido donde el estudiante crea conveniente. Las credenciales no deberán ser enviadas a los servicios. 

El web server deberá  retornar _404 Not Found_ para cualquier otro _path_ no existente.

A modo de simplificación, usaremos sólo _HTTP_, pero aclarando que esto posee *graves problemas de seguridad*.
Ambos servicios deben estar configurados con _SystemD_ para soportar los comandos, _restart_, _reload_, _stop_, _start_ y deberán ser inicializados de manera automática cuando el sistema operativo _botee_.

Ambos servicios deber _logear_ todas sus peticiones con el siguiente formato:

```sh
    <Timestamp>|<Nombre Del Servicio>| <Mensaje>
```

El _<Mensaje>_ sera definido por cada una de las acciones de los servicios.

El gráfico \ref{fig:arq} se describe la arquitectura requerida.

\begin{figure}
    \centering
    \includegraphics[width=14cm]{WebAppReferenceArchitecture}
    \label{fig:arq}
    \caption{Arquitectura del sistema}
\end{figure}


A continuación, detallaremos los dos servicios a crear y las funcionalidades de cada uno.

### Servicio de Usuarios
Este servicio se encargará de crear usuarios y listarlos. Estos usuarios deberán poder _logearse_ vía _SSH_ luego de su creación.

#### POST /api/users
Endpoints para la creación de usuario en el sistema operativo:

```C
    POST http://{{server}}/api/users
    \end{lstlisting}
    Request
    \begin{lstlisting}[language=C]
        curl --request POST \
            --url http:// {server}}/api/users \
            -u USER:SECRET \
            --header 'accept: application/json' \
            --header 'content-type: application/json' \
            --data '{"username": "myuser", "password": "mypassword"}'
    \end{lstlisting}
    Respuesta
    \begin{lstlisting}
        {
            "id": 142,
            "username": "myuser",
            "created_at": "2019-06-22 02:19:59"
        }
    \end{lstlisting}
```
El _<Mensaje>_ para el log será: _Usuario <Id> creado_
  
#### GET /api/users
Endpoint para obtener todos los usuario del sistema operativo y sus identificadores.
```C
    curl --request GET \
        --url http://{{server}}/api/users \
        -u USER:SECRET \
        --header 'accept: application/json' \
        --header 'content-type: application/json'
```
Respuesta
```C
    {
      "data": [
          {
              "user_id": 2,
              "username": "user1",  
          },
          {
              "user_id": 1,
              "username": "user2"
          },
          ...
      ]
    }
```
El  _<Mensaje>_ para el log será:  _Usuario listados: <cantidad de usuario del SO>_
### Servicio de estado
Este servicio devolverá el estado actual del sistema

*REQUEST /api/servers/hardwareinfo*

```C
    curl --request GET \
        --url http://{{server}}/api/servers/hardwareinfo \
        -u USER:SECRET \
        --header 'accept: application/json' \
        --header 'content-type: application/json'
```

Respuesta
```C
    {
        "kernelVersion": "4.4.0-87-generic",
        "processorName": "Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz",
        "totalCPUCore": 2,
        "totalMemory": 0.9520301818847656,
        "freeMemory": 0.3449363708496094,
        "diskTotal": 9.376751616,
        "diskFree": 5.573582848,
        "loadAvg": 0,
        "uptime": "74h 54m 7s"
    }
```
El _<Mensaje>_ para el log será: _Estadísticas requeridas desde el _<host>_.
El _<host>_ aquí, debe ser la dirección IP del cliente que realice el _request_.
Para obtenerlo, se puede pasar por parámetro desde web server al servicio mediante un _header_.
  
 
## Entrega
Se deberá proveer los archivos fuente, así como cualquier otro archivo asociado a la compilación, archivos de proyecto "Makefile" y el código correctamente documentado, todo en el repositorio, donde le Estudiante debe demostrar avances semana a semana mediante _commits_.

También se debe entregar un informe, guia tipo _How to_, explicando paso a paso lo realizado (puede ser un _Markdown_). El informe además debe contener el diseño de la solución con una explicacion detallada de la misma. Se debe asumir que las pruebas de compilación se realizarán en un equipo que cuenta con las herramientas típicas de consola para el desarrollo de programas (Ejemplo: gcc, make), y NO se cuenta con herramientas "GUI" para la compilación de los mismos (Ej: eclipse).

El install del makefile deberá copiar los archivos de configuración de systemd para poder luego ser habilitados y ejecutados por linea de comando.
El script debe copiar los archivos necesarios para el servicio Nginx systemd para poder luego ser habilitados y ejecutados por linea de comando.
Los servicios deberán pasar una batería de test escritas en _postman_ provistas. TBD.


## Evaluación
El presente trabajo práctico es individual deberá entregarse antes del jueves 4 de junio de 2020 a las 23:55 mediante el LEV.  Será corregido y luego deberá coordinar una fecha para la defensa oral del mismo.

## Referencias y ayudas
[Systrem D ](https://systemd.io/)
[System D en Freedesktop](https://www.freedesktop.org/wiki/Software/systemd/)
[nginx](https://docs.nginx.com/)
[Ulfius HTTP Framework](https://github.com/babelouest/ulfius)
[Kore Web PLataform](https://kore.io/)
