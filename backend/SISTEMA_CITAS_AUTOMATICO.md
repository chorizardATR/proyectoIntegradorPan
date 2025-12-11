# Sistema de Asignación Automática de Citas y Vencimiento

## 📋 Resumen de Implementación

### 1. ✅ Asignación Automática de Asesores

**Cómo funciona:**
- Cuando se crea una cita **sin especificar un asesor** (campo vacío), el sistema asigna automáticamente al asesor que tiene **menos citas activas**.
- Solo se cuentan citas en estados: `Programada`, `Confirmada`, `Reprogramada`
- **NO se cuentan**: citas `Vencida`, `Cancelada`, `Realizada`, `No asistió`

**Algoritmo:**
```python
def asignar_asesor_automaticamente(supabase):
    1. Obtener todos los usuarios (asesores)
    2. Obtener todas las citas activas (estados: Programada, Confirmada, Reprogramada)
    3. Contar cuántas citas activas tiene cada asesor
    4. Asignar al asesor con el menor número de citas activas
```

**Ejemplo:**
- Asesor A: 5 citas activas
- Asesor B: 3 citas activas + 2 vencidas = **3 citas activas**
- Asesor C: 4 citas activas
- **Resultado**: Se asigna al Asesor B (tiene 3, el menor número)

**En el Frontend:**
- El selector de asesor muestra: `🤖 Asignar automáticamente (asesor con menos citas)`
- Si se selecciona esta opción (valor vacío), el backend asigna automáticamente
- También se puede elegir manualmente un asesor específico

---

### 2. ✅ Actualización Automática de Citas Vencidas

**Cómo funciona:**
- Cada vez que se **lista las citas** (GET /citas-visita/), el sistema ejecuta automáticamente `actualizar_citas_vencidas()`
- Esta función busca todas las citas con estados `Programada`, `Confirmada` o `Reprogramada` cuya fecha ya pasó
- Las actualiza automáticamente a estado `Vencida`

**Algoritmo:**
```python
def actualizar_citas_vencidas(supabase):
    1. Obtener fecha/hora actual
    2. Buscar citas con estado Programada/Confirmada/Reprogramada
    3. Filtrar las que tienen fecha_visita_cita < ahora
    4. Actualizar cada una a estado 'Vencida'
```

**Estados de Citas:**
- **Programada**: Cita agendada, pendiente de confirmación
- **Confirmada**: Cliente confirmó que asistirá
- **Reprogramada**: Se cambió la fecha original
- **Realizada**: La visita se completó exitosamente
- **Cancelada**: Se canceló antes de la fecha
- **No asistió**: El cliente no llegó a la cita
- **Vencida**: ⏰ La fecha pasó y no se realizó (automático)

---

### 3. 📝 Cambios en el Código

#### Backend (`Backend/app/routes/citas_visita.py`)

**Nuevas funciones:**
```python
# Función 1: Actualizar citas vencidas
def actualizar_citas_vencidas(supabase):
    """Marca como 'Vencida' las citas pasadas que no se realizaron"""

# Función 2: Asignar asesor automáticamente
def asignar_asesor_automaticamente(supabase):
    """Asigna al asesor con menos citas activas"""
```

**Cambios en el endpoint POST /citas-visita/:**
```python
# Antes: id_usuario_asesor era obligatorio
# Ahora: es opcional, se asigna automáticamente si está vacío

if not cita.id_usuario_asesor:
    asesor_id = asignar_asesor_automaticamente(supabase)
    cita.id_usuario_asesor = asesor_id
```

**Cambios en el endpoint GET /citas-visita/:**
```python
# Se agregó al inicio:
actualizar_citas_vencidas(supabase)
# Esto se ejecuta ANTES de listar las citas
```

#### Schema (`Backend/app/schemas/cita_visita.py`)

**Cambio:**
```python
# Antes:
id_usuario_asesor: str  # Obligatorio

# Ahora:
id_usuario_asesor: Optional[str] = None  # Opcional
```

#### Frontend (`Frontend/FrontendAdmin/src/pages/citas/CitaForm.jsx`)

**Cambios:**
1. Selector de asesor actualizado con texto explicativo
2. Validación removida para `id_usuario_asesor` (ya no es obligatorio)
3. Agregado estado "Vencida" al selector de estados
4. Tooltips explicativos sobre el funcionamiento automático

---

### 4. 🧪 Cómo Probarlo

#### Prueba 1: Asignación Automática
1. Ve a **Citas > Nueva Cita**
2. Llena los campos obligatorios
3. **Deja el selector de "Asesor" en "Asignar automáticamente"**
4. Guarda la cita
5. Verifica que se asignó al asesor con menos citas activas

#### Prueba 2: Vencimiento Automático
1. Crea una cita con fecha en el pasado (en modo edición)
2. O espera a que pase la fecha de una cita programada
3. Actualiza la lista de citas (F5)
4. La cita debe aparecer con estado "Vencida" automáticamente

#### Prueba 3: Balance de Carga
1. Crea varias citas sin especificar asesor
2. Verifica que se distribuyen equitativamente entre los asesores
3. El que tiene menos citas siempre recibe la siguiente

---

### 5. 🔍 Verificación Manual en Base de Datos

**Contar citas activas por asesor:**
```sql
SELECT 
    id_usuario_asesor,
    COUNT(*) as citas_activas
FROM citavisita
WHERE estado_cita IN ('Programada', 'Confirmada', 'Reprogramada')
GROUP BY id_usuario_asesor
ORDER BY citas_activas ASC;
```

**Ver citas vencidas:**
```sql
SELECT 
    id_cita,
    fecha_visita_cita,
    estado_cita
FROM citavisita
WHERE estado_cita = 'Vencida';
```

**Actualizar manualmente citas vencidas (si es necesario):**
```sql
UPDATE citavisita
SET estado_cita = 'Vencida'
WHERE estado_cita IN ('Programada', 'Confirmada', 'Reprogramada')
  AND fecha_visita_cita < NOW();
```

---

### 6. ⚠️ Consideraciones Importantes

**Zona Horaria:**
- El sistema usa UTC internamente
- Asegúrate de que las fechas se manejen correctamente con timezone

**Performance:**
- La función `actualizar_citas_vencidas()` se ejecuta en cada listado
- Si hay muchas citas, considera optimizar con un job programado (cron)

**Lógica de Negocio:**
- Una cita "Vencida" NO se cuenta para asignación de nuevas citas
- Solo se cuentan citas "activas" que requieren atención del asesor

**Estados Finales vs. Activos:**
- **Estados Activos** (requieren atención): Programada, Confirmada, Reprogramada
- **Estados Finales** (no requieren atención): Realizada, Cancelada, No asistió, Vencida

---

### 7. 📊 Flujo Completo

```
Usuario crea cita sin asesor
    ↓
Backend recibe id_usuario_asesor = null
    ↓
asignar_asesor_automaticamente()
    ↓
Cuenta citas activas por asesor
    ↓
Asigna al que tiene menos
    ↓
Cita creada con asesor asignado
    ↓
    ↓
(El tiempo pasa...)
    ↓
Usuario lista citas
    ↓
actualizar_citas_vencidas()
    ↓
Marca citas pasadas como "Vencida"
    ↓
Lista actualizada mostrada
```

---

## ✅ Conclusión

El sistema ahora:
1. ✅ Asigna automáticamente citas al asesor con menos carga de trabajo
2. ✅ Actualiza automáticamente citas vencidas
3. ✅ Balancea la carga entre asesores equitativamente
4. ✅ Mantiene estados actualizados sin intervención manual
5. ✅ Ignora citas vencidas al calcular carga de trabajo

Todo funciona de forma transparente y automática para el usuario. 🎉
