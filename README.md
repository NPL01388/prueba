# NPL Domicilios: app de pedidos + app del mensajero

Dos vistas web adaptables para celular, conectadas a Supabase y listas para publicar en Vercel.

## Qué hace

- **Tomar pedidos** (`/pedidos`): crea un pedido compartido y avisa al mensajero.
- **Mensajero** (`/mensajero`): recibe cambios en tiempo real, puede aceptar, iniciar recorrido, abrir la dirección en mapas, contactar por WhatsApp y confirmar la entrega con el código de seguridad.
- La notificación push puede aparecer con la app en segundo plano o cerrada, una vez que el mensajero inicia sesión, instala/abre la app en un contexto compatible y acepta los avisos.
- Los pedidos se guardan en Supabase. Vercel sirve las dos apps y ejecuta funciones seguras para crear pedidos, registrar dispositivos y cambiar estados.

## Puesta en marcha

1. Crea un proyecto en Supabase y activa Email/Password en **Authentication > Providers**. Desactiva el registro público; crea desde el panel las cuentas del administrador y del mensajero.
2. Copia y ejecuta `supabase/schema.sql` en **SQL Editor**.
3. En **Authentication > Users**, copia el UUID de cada usuario. Añade una fila en `public.profiles` para cada uno, asignando `manager` al administrador y `courier` al mensajero. Guarda su nombre en `full_name`.
4. En Supabase, copia Project URL y la clave publicable/anon. En Vercel, importa esta carpeta como proyecto y configura `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` y `SUPABASE_URL`.
5. Configura `SUPABASE_SERVICE_ROLE_KEY` en Vercel como variable privada. **No la pongas en archivos públicos ni la compartas.**
6. Genera un par de claves VAPID para notificaciones web y añade `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` a Vercel. Además pon la clave pública en `VITE_VAPID_PUBLIC_KEY` para que Vite la incluya en la app del mensajero. Después de cambiar variables, vuelve a desplegar.
7. Abre `/mensajero` desde el celular del mensajero, inicia sesión y pulsa **Activar avisos**. En iPhone, añade la web a la pantalla de inicio y ábrela desde ese icono antes de permitir avisos. Mantén `/pedidos` para tomar pedidos.

Para generar las claves VAPID, ejecuta `npx web-push generate-vapid-keys` en una terminal local. Copia los valores directamente a las variables de entorno de Vercel; no los pegues en este chat.

## Variables de Vercel

| Variable | Dónde se usa | Privada |
|---|---|---|
| `VITE_SUPABASE_URL` | Navegador | No |
| `VITE_SUPABASE_ANON_KEY` | Navegador, protegida por RLS | No |
| `VITE_VAPID_PUBLIC_KEY` | Navegador | No |
| `SUPABASE_URL` | Funciones API | No |
| `SUPABASE_SERVICE_ROLE_KEY` | Funciones API | **Sí** |
| `VAPID_PUBLIC_KEY` | Funciones API | No |
| `VAPID_PRIVATE_KEY` | Funciones API | **Sí** |
| `VAPID_SUBJECT` | Funciones API | No |

El proyecto está hecho como una base lista para configurar y desplegar. Las notificaciones necesitan permisos del navegador y una conexión a internet. No se pueden activar desde el HTML local ni sin configurar Supabase/Vercel.

Al crear un pedido, la app muestra el código de entrega privado y permite enviarlo al cliente por WhatsApp. El mensajero solo ve una referencia del pedido; la app valida el código contra una tabla privada del servidor al confirmar la entrega.

## Publicación en Vercel

Sube el contenido de esta carpeta a un repositorio privado de GitHub y, desde Vercel, importa ese repositorio. Usa el comando de compilación `npm run build` y la carpeta de salida `dist`. Añade las variables de entorno de la tabla para Production y vuelve a desplegar. La carpeta `api/` se publica como funciones de Vercel.
