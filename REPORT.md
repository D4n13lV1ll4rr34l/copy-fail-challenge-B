# Reporte técnico — CVE-2026-31431

El bug raíz de CVE-2026-31431 se encuentra en el archivo crypto/algif_aead.c, específicamente en la función `aead_recvmsg(). El problema fue introducido por una optimización implementada en 2017 relacionada con operaciones AEAD dentro de la interfaz AF_ALG del kernel Linux.

La vulnerabilidad ocurre pq el kernel reutiliza el mismo scatterlist tanto para lectura como para escritura durante las operaciones que usa, por esto mismo pasa que provoca que ciertas escrituras terminen afectando páginas pertenecientes al page cache del sistema. Un atacante local puede modificar memoria cacheada asociada a archivos privilegiados sin alterar el contenido persistente en disco.

El write realizado es peligroso porque puede terminar escribiendo fuera de los límites esperados del buffer criptográfico. Debido al uso compartido de scatterlists, el destino puede apuntar indirectamente a páginas cacheadas de binarios setuid-root.

El exploit es considerado “stealthy” porque no modifica directamente el archivo almacenado en disco. El ataque solo altera temporalmente la versión presente en el page cache del kernel. Como resultado, verificaciones de integridad tradicionales pueden seguir mostrando el archivo original intacto, mientras el kernel ejecuta una versión modificada en memoria.

Este laboratorio nos permite revisar conceptos importantes de sistemas operativos como page cache, permisos setuid, inodos y separación entre almacenamiento persistente y memoria RAM. Aunque el archivo físico permanezca intacto, el kernel termina ejecutando la versión alterada cargada en memoria, lo que llega a afectar DE VERDAD.

También revisé y comprobé cómo múltiples cambios aparentemente razonables pueden combinarse para crear una vulnerabilidad crítica. La optimización de rendimiento introducida en 2017 parecía válida individualmente, pero terminó permitiendo una primitive de escritura extremadamente peligrosa dentro del kernel Linux.


