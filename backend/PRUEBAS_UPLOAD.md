# 🧪 PRUEBAS DEL ENDPOINT DE SUBIDA DE IMÁGENES

## ✅ Estado del Backend
- **Servidor:** ✅ Corriendo en `http://localhost:8000`
- **Endpoint nuevo:** `POST /api/imagenes-propiedad/upload/{id_propiedad}`

---

## 📋 OPCIÓN 1: Probar con Thunder Client / Postman

### Paso 1: Login para obtener token
```
POST http://localhost:8000/api/login
Content-Type: application/x-www-form-urlencoded

username=admin
password=admin123
```

**Respuesta:**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "token_type": "bearer",
  "user": {...}
}
```

Copia el `access_token`.

---

### Paso 2: Subir imágenes

```
POST http://localhost:8000/api/imagenes-propiedad/upload/1
Authorization: Bearer {TU_TOKEN_AQUI}
Content-Type: multipart/form-data

Body (form-data):
- Key: imagenes | Type: File | Value: [Selecciona una o más imágenes]
- Key: imagenes | Type: File | Value: [Otra imagen]
- Key: imagenes | Type: File | Value: [Otra imagen]
```

**Respuesta esperada:**
```json
{
  "mensaje": "✅ 3 imágenes subidas exitosamente",
  "propiedad_id": 1,
  "imagenes": [
    {
      "id_imagen": 123,
      "id_propiedad": 1,
      "url_imagen": "/uploads/propiedades/1/abc-123.jpg",
      "descripcion_imagen": "Imagen 1 - Subida por admin el 2025-10-22 15:30",
      "es_portada_imagen": true,
      "orden_imagen": 0
    },
    {...}
  ],
  "portada": "/uploads/propiedades/1/abc-123.jpg"
}
```

---

### Paso 3: Ver la imagen

Abre en el navegador:
```
http://localhost:8000/uploads/propiedades/1/abc-123.jpg
```

---

## 📋 OPCIÓN 2: Probar con el script Python

Asegúrate de tener una propiedad con ID=1 en tu base de datos, luego ejecuta:

```bash
cd C:\Users\USUARIO\Downloads\ProyectoInt\Backend
C:/Users/USUARIO/Downloads/ProyectoInt/.venv/Scripts/python.exe test_upload_imagenes.py
```

**Nota:** Modifica el script si necesitas cambiar:
- Usuario/contraseña (líneas 13-14)
- ID de propiedad (línea 11)

---

## 🔍 Verificar en Base de Datos

Consulta en Supabase:
```sql
SELECT * FROM imagenpropiedad WHERE id_propiedad = 1 ORDER BY orden_imagen;
```

Deberías ver las imágenes registradas con sus URLs.

---

## 📂 Verificar Carpeta Uploads

Navega a:
```
C:\Users\USUARIO\Downloads\ProyectoInt\Backend\uploads\propiedades\1\
```

Deberías ver los archivos de imagen guardados con nombres tipo `abc123-def456.jpg`.

---

## 🚀 PRÓXIMO PASO: App Móvil React Native

Una vez verificado que el endpoint funciona, el siguiente paso es:

1. **Configurar proyecto Expo** en `Frontend/FrontendAsesor`
2. **Crear pantalla de login** (reutilizando lógica de FrontendAdmin)
3. **Crear pantalla de cámara** para tomar/seleccionar fotos
4. **Implementar upload** usando FormData desde React Native

---

## 💡 Código de ejemplo para React Native

```javascript
// En tu servicio de imágenes (Frontend/FrontendAsesor/src/api/imagenService.js)
import axios from './axios';

export const subirImagenes = async (propiedadId, imagenes) => {
  const formData = new FormData();
  
  imagenes.forEach((imagen, index) => {
    const filename = imagen.uri.split('/').pop();
    const match = /\.(\w+)$/.exec(filename);
    const type = match ? `image/${match[1]}` : 'image/jpeg';
    
    formData.append('imagenes', {
      uri: imagen.uri,
      name: filename,
      type: type,
    });
  });
  
  const response = await axios.post(
    `/imagenes-propiedad/upload/${propiedadId}`,
    formData,
    {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    }
  );
  
  return response.data;
};
```

---

## ✅ Checklist de Implementación

Backend:
- [x] Endpoint `/imagenes-propiedad/upload/{id_propiedad}` creado
- [x] Soporte para `multipart/form-data` configurado
- [x] Carpeta `uploads/` configurada
- [x] Archivos estáticos servidos con `/uploads`
- [ ] Probar endpoint manualmente
- [ ] Verificar imágenes guardadas

App Móvil (Próximo):
- [ ] Configurar proyecto Expo
- [ ] Implementar login
- [ ] Implementar cámara
- [ ] Implementar upload de imágenes
- [ ] Probar flujo completo

---

## 🎯 ¿Qué sigue?

**Si el endpoint funciona correctamente:**
1. ✅ Backend listo para recibir imágenes
2. 🚀 Empezar con la app móvil Expo

**¿Quieres que te ayude a:**
- A) Configurar el proyecto Expo ahora
- B) Probar más el endpoint primero
- C) Ver la documentación interactiva en http://localhost:8000/docs

**Avísame qué prefieres hacer!** 😊
