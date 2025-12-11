# Configuración de Supabase Storage para Documentos de Propiedades

## 📦 Crear Bucket

Debes crear un bucket en Supabase Storage siguiendo estos pasos:

### 1. Acceder a Supabase Dashboard
- Ve a https://supabase.com/dashboard
- Selecciona tu proyecto
- En el menú lateral, ve a **Storage**

### 2. Crear el Bucket
1. Haz clic en **"New bucket"**
2. Configuración del bucket:
   - **Name**: `documentos-propiedades`
   - **Public bucket**: ✅ **SÍ** (marcar como público)
   - **File size limit**: `10 MB` (opcional, para limitar tamaño de archivos)
   - **Allowed MIME types**: `application/pdf, application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document, text/plain, image/jpeg, image/png` (opcional)

3. Haz clic en **"Create bucket"**

### 3. Configurar Políticas de Seguridad (Policies)

**OPCIÓN A: Políticas Públicas (Más Simple - Recomendado)**

Una vez creado el bucket, ve a **Storage > Policies** y crea las siguientes políticas públicas:

#### Política 1: Permitir TODAS las operaciones públicas
1. Haz clic en **"New Policy"** en el bucket `documentos-propiedades`
2. Selecciona **"For full customization"** o **"Custom"**
3. Llena los campos:
   - **Policy name**: `Permitir todas las operaciones públicas`
   - **Allowed operation**: Marca **ALL** (o marca INSERT, SELECT, UPDATE, DELETE)
   - **Target roles**: `public`
   - **Policy definition (USING)** (pega esto):
   ```sql
   true
   ```
   - **Policy definition (WITH CHECK)** (pega esto):
   ```sql
   true
   ```
4. Haz clic en **"Review"** y luego **"Save policy"**

Esta única política permite todas las operaciones (subir, ver, eliminar) de forma pública. Es más simple y funciona con tu sistema de autenticación JWT local.

---

**OPCIÓN B: Si prefieres más control (Alternativa)**

Si OPCIÓN A no funciona, elimina esa política y crea estas 3 por separado:

#### Política 1: INSERT público
- **Policy name**: `Public Insert`
- **Policy command**: `INSERT`
- **Target roles**: `public`
- **WITH CHECK**: `true`

#### Política 2: SELECT público  
- **Policy name**: `Public Select`
- **Policy command**: `SELECT`
- **Target roles**: `public`
- **USING**: `true`

#### Política 3: DELETE público
- **Policy name**: `Public Delete`
- **Policy command**: `DELETE`
- **Target roles**: `public`
- **USING**: `true`

> **⚠️ IMPORTANTE**: 
> - Solo pon `true` en los campos USING o WITH CHECK, no copies código SQL completo
> - El backend usa la service role key que bypasea RLS, pero estas políticas son necesarias para acceso desde el navegador
> - Como usas JWT local (no Supabase Auth), las políticas deben ser `public` no `authenticated`

### 4. Agregar Campo a la Tabla (si no existe)

Verifica que la tabla `documentopropiedad` tenga el campo `nombre_archivo_original`:

```sql
-- Agregar columna si no existe
ALTER TABLE documentopropiedad 
ADD COLUMN IF NOT EXISTS nombre_archivo_original TEXT;
```

## 🔍 Verificar Configuración

1. Ve a Storage > Policies en Supabase
2. Deberías ver las 3 políticas creadas para el bucket `documentos-propiedades`
3. Verifica que el bucket sea público (icono de globo)

## 📝 Tipos de Archivos Permitidos

El sistema acepta los siguientes formatos:
- **PDF**: `.pdf`
- **Word**: `.doc`, `.docx`
- **Texto**: `.txt`
- **Imágenes**: `.jpg`, `.jpeg`, `.png`

**Límite de tamaño**: 10 MB por archivo

## 🚀 Probar el Sistema

1. Ve a **Propiedades** en el sistema
2. Haz clic en el ícono de ojo (👁️) en cualquier propiedad
3. Haz clic en **"Subir Documento"**
4. Selecciona un archivo y súbelo
5. El archivo debería aparecer en la lista de documentos

## 🔧 Solución de Problemas

### Error: "Error al subir archivo a storage"
- Verifica que el bucket exista
- Verifica que el bucket sea público
- Verifica las políticas de seguridad

### Error: "Error al eliminar documento"
- Verifica la política DELETE
- Verifica que el usuario esté autenticado

### Los archivos no se ven
- Verifica que el bucket sea público
- Verifica la política SELECT (debe ser para `public`)

## 📂 Estructura de Archivos en Storage

Los archivos se organizan de la siguiente manera:
```
documentos-propiedades/
  ├── {id_propiedad_1}/
  │   ├── 20250107_143022_a3b4c5d6_titulo.pdf
  │   └── 20250107_150130_f7e8d9c0_plano.pdf
  ├── {id_propiedad_2}/
  │   └── 20250107_160245_1a2b3c4d_contrato.docx
  └── ...
```

Cada archivo tiene:
- **Fecha y hora**: `YYYYMMDD_HHMMSS`
- **UUID corto**: `8 caracteres aleatorios`
- **Nombre original**: El nombre del archivo subido

Esto previene colisiones de nombres y organiza los archivos por propiedad.
