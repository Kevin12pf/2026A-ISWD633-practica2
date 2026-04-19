# Mi Aprendizaje y Conclusiones

**Principales aprendizajes logrados:**

1. **Manejo de Variables de Entorno (`-e`):** Aprendí que muchos servicios (como MySQL o Postgres) requieren configuraciones obligatorias inyectadas directamente durante la creación del contenedor mediante variables de entorno para temas críticos, como definir contraseñas raíz o usuarios.
2. **Redes en Docker (`bridge`):** Comprendí cómo aislar y conectar servicios en redes bridge personalizadas. Ahora sé que vincular contenedores a una misma red les permite resolverse y comunicarse de manera interna y segura sin necesidad de exponer puertos al servidor anfitrión para todo.
3. **Persistencia y el concepto "Stateless":** El ejercicio final de WordPress me permitió evidenciar de primera mano qué significa que un contenedor sea efímero. Al eliminar el contenedor web de WordPress y volver a crearlo, me di cuenta de que todos mis datos seguían intactos, ya que todo el estado real del sitio reside en el contenedor de base de datos MySQL (que no había sido eliminado).

**Resolución de problemas presentados:**
Durante la práctica experimenté un inconveniente al intentar abrir la página de `pgAdmin` en mi navegador (el puerto no respondía). Al analizar la situación y revisar los contenedores activos usando `docker ps`, me di cuenta de que únicamente había levantado el servidor de base de datos, pero olvidé iniciar el servidor web gráfico del cliente (`pgAdmin`). Ambos problemas se solucionaron asegurándome de levantar los servicios correspondientes y corrigiendo la sintaxis en la línea de comandos de la terminal.

---

### Consulta: Cómo se gestionan datos confidenciales con los secretos de Docker (Docker Secrets)

Los **Docker Secrets** son un mecanismo nativo diseñado para gestionar, almacenar y distribuir de forma segura datos confidenciales, tales como contraseñas, tokens de API, certificados TLS, y llaves SSH. A diferencia de las variables de entorno, que pueden ser expuestas fácilmente si alguien revisa los logs o hace un `docker inspect` (como vimos al inspeccionar la IP en esta práctica), los Secrets proveen mayor seguridad.

**¿Cómo funcionan?**

- Los secretos únicamente están disponibles de forma nativa cuando se utiliza **Docker Swarm** (el orquestador nativo de Docker).
- Cuando añades un secreto a Docker Swarm, este se cifra durante el tránsito sobre la red y en el almacenamiento de los nodos gestores (managers).
