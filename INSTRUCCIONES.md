# Kit del taller: EC2 con GitHub Actions y CloudFormation

Este kit ya fue probado en una cuenta real de AWS y funciona. Sigue los pasos en orden.

> Este archivo es solo para ti. No es necesario subirlo al repositorio.

## Contenido

```
.github/workflows/deploy.yml   despliega en cada push a main (no hay que modificarlo)
.github/workflows/destroy.yml  borra todo con un botón al terminar (no hay que modificarlo)
templates/template.yaml        plantilla de CloudFormation (aquí personalizas tu página)
README.md                      descripción del proyecto (pon tus datos y la IP al final)
```

## Reglas para que no falle

1. **Nunca crees claves de acceso en el usuario raíz.** Si AWS muestra el aviso amarillo
   "No se recomiendan las claves de acceso de los usuarios raíz", dale **Cancelar**.
2. La plantilla usa **t3.micro**. No la cambies a t2.micro: en cuentas nuevas no es de capa
   gratuita y falla con "not eligible for Free Tier".
3. Si el primer despliegue falla, corrige y vuelve a hacer push. El workflow borra solo el
   stack fallido (ROLLBACK_COMPLETE) antes de desplegar.

## Paso 1. Crear el repositorio

En GitHub crea un repositorio con el nombre `CODIGO-nombre-apellido-iac-aws`
(ejemplo: `123456-wilder-duarte-iac-aws`).

## Paso 2. Crear un usuario IAM para GitHub (cuenta normal de AWS)

1. En AWS busca **IAM** → **Usuarios** → **Crear usuario**.
2. Nombre: `github-actions-taller-<tuapellido>`. No marques acceso a la consola. **Siguiente**.
3. **Adjuntar políticas directamente** y marca:
   - `AWSCloudFormationFullAccess`
   - `AmazonEC2FullAccess`
   - `AmazonSSMReadOnlyAccess`
4. **Siguiente** → **Crear usuario**.
5. Abre el usuario → **Credenciales de seguridad** → **Crear clave de acceso**.
6. Caso de uso **Interfaz de línea de comandos (CLI)** → marca la confirmación →
   **Siguiente** → **Crear clave de acceso**.
7. Descarga el **.csv** (la clave secreta no se vuelve a mostrar) → **Listo**.

**Si usas AWS Academy (Learner Lab):** no crees usuario IAM. Haz **Start Lab** →
**AWS Details** → **AWS CLI: Show** y copia los tres valores: `aws_access_key_id`,
`aws_secret_access_key` y `aws_session_token`. Vencen cuando se apaga el lab.

## Paso 3. Crear los secretos en GitHub

Repositorio → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**:

| Nombre del secreto | Valor |
| --- | --- |
| `AWS_ACCESS_KEY_ID` | Clave de acceso del usuario IAM |
| `AWS_SECRET_ACCESS_KEY` | Clave secreta del usuario IAM |
| `AWS_SESSION_TOKEN` | Solo con AWS Academy |

Nunca escribas las claves dentro de los archivos ni las muestres en capturas.

## Paso 4. Personalizar y subir los archivos

1. Abre `templates/template.yaml` y cambia en el HTML: `TU NOMBRE`, `TU NOMBRE COMPLETO`,
   `TU_CODIGO` y el enlace `TU_USUARIO/TU_REPOSITORIO`. Puedes cambiar colores, textos y
   agregar imágenes o enlaces, siempre entre `<<'EOF'` y `EOF`.
2. Pon tus datos en `README.md`.
3. Sube los archivos a la rama `main` respetando las carpetas. En GitHub web:
   **Add file** → **Upload files** y arrastra el contenido del kit (las carpetas
   `.github` y `templates` y el `README.md`).
   - Ojo: la carpeta `.github` empieza con punto y en algunos sistemas está oculta.
     Si no aparece, créala a mano con **Add file** → **Create new file** escribiendo
     `.github/workflows/deploy.yml` como nombre y pegando el contenido.

## Paso 5. Ver el despliegue

1. Pestaña **Actions** → abre la ejecución en curso. Tarda unos 2 minutos.
2. El último paso, "Mostrar las salidas del stack", muestra la `WebsiteURL`.
3. Abre `http://IP` (con http, no https). Si no carga, espera 1 o 2 minutos:
   Apache se está instalando.
4. En AWS: **CloudFormation** → stack `proyecto-web-iac` en **CREATE_COMPLETE**, y
   **EC2** → instancia `EC2-Clase-CF` en **En ejecución**.

## Paso 6. Modificaciones (sección 8 del taller)

User Data solo se ejecuta la primera vez que arranca la instancia. Para ver una página nueva:

1. Cambia el HTML (colores, textos, un enlace o una imagen).
2. Cambia las etiquetas (Tags), por ejemplo `Value: EC2-TuNombre-v2`.
3. Renombra el recurso `MyInstanceClase` a `MyInstanceClaseV2` en **Resources** y en los
   **tres Outputs** (`!Ref`, `!GetAtt` y `!Sub`).
4. Commit y push. El stack pasa a **UPDATE_COMPLETE** y la página queda en una IP nueva.

Notas:
- Cambiar el `Default` del parámetro `InstanceDescription` no cambia el título en un stack
  que ya existe. Cambia el texto directamente en el HTML.
- Para cambiar solo el README sin lanzar un despliegue, agrega `[skip ci]` al mensaje del commit.

## Al terminar: borrar todo desde GitHub

1. Toma todas las capturas de evidencia.
2. En GitHub: pestaña **Actions** → en la izquierda, **Eliminar infraestructura AWS** →
   botón **Run workflow** → escribe `ELIMINAR` (en mayúsculas) → **Run workflow**.
3. En unos 2 minutos el stack se borra junto con la EC2 y el Security Group.
   Si escribes otra palabra, el workflow no hace nada (es una protección contra clics por error).
4. Comprueba en AWS → **CloudFormation** que `proyecto-web-iac` ya no aparezca, y en **EC2**
   que la instancia esté **Terminada**.
5. Cuidado: el siguiente push a `main` vuelve a crear todo. Si ya terminaste, no hagas más push.
6. En IAM, desactiva la clave del usuario `github-actions-taller-<tuapellido>`.

También puedes borrarlo a mano: **CloudFormation** → selecciona `proyecto-web-iac` → **Eliminar**.
