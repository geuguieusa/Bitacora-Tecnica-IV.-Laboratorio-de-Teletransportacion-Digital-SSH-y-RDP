### Bitácora Técnica-IV. Laboratorio de Teletransportación Digital SSH y RDP

**Alumno:** Guillermo Eugui Sánchez | **Módulo:** Sistemas Informáticos | **Profesor:** Willman Acosta Lugo | **Fecha:** 08/05/2026

He levantado contenedores con Docker Compose: una es un servidor SSH (puerto 2222) y un escritorio remoto Ubuntu accesible por el navegador (localhost:3000/) y RDP (puerto 3389).

---

## SSH: Forjando la Llave Maestra

Generé las keys en Windows y las copie usando:

```
ssh-keygen -t ed25519 -C "guillermoeugui.25@campuscamara.es"
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh alumno@localhost -p 2222 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh"
```

Con esto pude hacer una conexión sin pedir contraseña.

Para el hardening entré como root con [ docker exec -it lab_ssh_servidor sh ] y edité .../etc/ssh/sshd_config: 

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

Recargué con `kill -HUP $(pgrep sshd)`.

---

## RDP: El Escritorio en tu Navegador

Accedí desde el navegador a http://localhost:3000 y vi el escritorio Ubuntu completo.

---

## Problemas encontrados

| Problema | Solución |
|---|---|
| alumno sin permisos sudo | Entrar como root con docker exec -it lab_ssh_servidor sh |
| Confusión entre CMD, PowerShell y terminal de VS Code | Usar siempre PowerShell |
| SSH ejecutado desde dentro del contenedor | Poner el comando ssh desde la máquina anfitriona, nunca desde el contenedor |
| kill -HUP $(cat /var/run/sshd.pid) me daba errores | Usar kill -HUP $(pgrep sshd) en su lugar |

---

## Reflexión — ¿Por qué SSH y no RDP en producción?

SSH no necesita de un entorno gráfico y consume muy pocos recursos y su autenticación por clave pública es mucho más segura que una contraseña. También permite automatizar tareas con scripts. RDP requiere escritorio, más ancho de banda y expone más superficie de ataque. En servidores Linux en la nube, SSH es el estándar.
