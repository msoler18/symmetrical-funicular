# Directrices de Desarrollo - Estándares de Ingeniería Élite

## Filosofía

### Creencias Fundamentales

- **Progreso incremental sobre grandes cambios** - Cambios pequeños y atómicos que compilan y pasan tests
- **Aprender del código existente** - Estudiar, entender patrones y planificar antes de implementar
- **Pragmático sobre dogmático** - Adaptarse a la realidad del proyecto manteniendo estándares
- **Intención clara sobre código ingenioso** - Ser aburrido, obvio y mantenible
- **Decisiones basadas en datos** - Medir impactos de rendimiento, no asumir
- **Fallar rápido, recuperarse más rápido** - Validación temprana y degradación elegante

### Manifiesto de Simplicidad

- Responsabilidad única por función/clase (máx. 30 líneas por función como guía)
- Evitar abstracciones prematuras - necesítalo 3 veces antes de abstraer
- Sin trucos ingeniosos - elige la solución aburrida
- Si necesitas explicarlo, es demasiado complejo
- Elimina código sin piedad - menos código = menos bugs

## Proceso

### 1. Planificación y Etapas

Divide trabajo complejo en 3-5 etapas. Documenta en `PLAN_IMPLEMENTACION.md`:

```markdown
## Etapa N: [Nombre]

**Objetivo**: [Entregable específico]
**Criterios de Éxito**: [Resultados medibles]
**Tests**: [Casos de prueba específicos]
**Dependencias**: [Qué debe existir antes]
**Riesgos**: [Qué podría salir mal]
**Plan de Rollback**: [Cómo deshacer si es necesario]
**Estado**: [No Iniciado|En Progreso|Completado|Bloqueado]
**Estimación de Tiempo**: [Horas/Días]
**Tiempo Real**: [Registrar para aprender]
```

Actualiza el estado según avances. Elimina el archivo cuando todas las etapas estén completas.

### 2. Flujo de Implementación

1. **Entender**

   - Estudiar patrones existentes en el código
   - Encontrar 3 implementaciones similares
   - Documentar suposiciones en PR/commit

2. **Diseñar**

   - Escribir interfaz/API primero
   - Obtener feedback temprano del enfoque
   - Considerar casos extremos desde el inicio

3. **Testear**

   - Escribir test primero
   - Incluir casos extremos
   - Benchmarks de rendimiento si es relevante

4. **Implementar**

   - Código mínimo para pasar (verde)
   - Seguir patrones existentes
   - Agregar telemetría/logging

5. **Refactorizar**

   - Limpiar con tests pasando
   - Optimizar solo con datos
   - Actualizar documentación

6. **Revisar**

   - Auto-revisión con ojos frescos
   - Ejecutar checklist
   - Probar manualmente una vez

7. **Commitear**
   - Commits atómicos con mensajes claros
   - Enlazar a issue/plan
   - Incluir "por qué" no solo "qué"

### 3. Protocolo Cuando Estás Atascado

**CRÍTICO**: Máximo 3 intentos por problema, luego PARA.

#### Después de Cada Intento Fallido:

1. **Documentar precisamente**:

   ```markdown
   ## Intento N

   **Hipótesis**: Lo que pensé que funcionaría
   **Implementación**: Lo que realmente hice
   **Resultado**: Error/fallo exacto
   **Aprendizaje**: Por qué no funcionó
   ```

2. **Después de 3 Intentos - PARADA OBLIGATORIA**:

   - Tomar break de 15 minutos mínimo
   - Escribir hallazgos en canal del equipo/issue
   - Considerar pair programming
   - Cuestionar si estás resolviendo el problema correcto

3. **Investigar alternativas**:

   - Encontrar 2-3 implementaciones similares
   - Revisar Stack Overflow con el error exacto
   - Revisar documentación del framework/librería
   - Buscar issues existentes

4. **Cuestionar fundamentos**:
   - ¿Es el nivel de abstracción correcto?
   - ¿Se puede dividir en problemas más pequeños?
   - ¿Existe un enfoque más simple?
   - ¿Estoy sobre-ingenierizando?

## Estándares Técnicos

### Principios de Arquitectura

- **Composición sobre herencia** - Usar inyección de dependencias e interfaces
- **Explícito sobre implícito** - Flujo de datos claro, sin magia
- **Inmutabilidad por defecto** - Mutar solo cuando sea necesario con propiedad clara
- **Asíncrono por diseño** - Operaciones no bloqueantes, cancelación apropiada
- **Estrategia de invalidación de caché** - Definir desde el inicio, no como idea tardía
- **Feature flags para cambios riesgosos** - Deploy != Release

### Métricas de Calidad de Código

- **Cada commit debe**:

  - [ ] Compilar exitosamente
  - [ ] Pasar todos los tests existentes
  - [ ] Incluir tests para nueva funcionalidad (objetivo 80% cobertura)
  - [ ] Pasar linting/formatting
  - [ ] Actualizar documentación relevante
  - [ ] No degradar rendimiento en >5%

- **Antes de commitear**:
  ```bash
  # Ejecutar este checklist
  make lint
  make test
  make bench  # si es sensible al rendimiento
  git diff --staged  # auto-revisión
  ```

### Estrategia de Manejo de Errores

```typescript
// Patrón de ejemplo
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number,
    public isOperational: boolean = true
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}
```

- Fallar rápido con mensajes descriptivos
- Incluir contexto para debugging (ID de request, ID de usuario, etc.)
- Distinguir errores operacionales vs de programación
- Registrar errores con severidad apropiada
- Nunca tragar excepciones silenciosamente
- Implementar circuit breakers para servicios externos

### Directrices de Rendimiento

- **Medir antes de optimizar** - Usar profilers, no intuición
- **Establecer presupuestos de rendimiento**:
  - Respuesta API: <200ms p50, <1s p99
  - Interacción frontend: <100ms
  - Tiempo de build: <2 minutos
- **Monitorear en producción**:
  - Herramientas APM requeridas
  - Métricas de usuarios reales (RUM)
  - Configurar alertas para degradación

## Framework de Decisiones

Cuando existen múltiples enfoques válidos, puntúa cada opción (1-5):

| Criterio       | Peso | Opción A | Opción B |
| -------------- | ---- | -------- | -------- |
| Testeabilidad  | 25%  |          |          |
| Legibilidad    | 20%  |          |          |
| Rendimiento    | 15%  |          |          |
| Consistencia   | 15%  |          |          |
| Simplicidad    | 15%  |          |          |
| Reversibilidad | 10%  |          |          |

**Elige el puntaje ponderado más alto, documenta decisión en ADR (Registro de Decisión de Arquitectura)**

## Integración al Proyecto

### Aprendiendo el Código Base

1. **Primer Día**:

   - Ejecutar suite de tests exitosamente
   - Desplegar en ambiente local
   - Leer últimos 10 PRs mergeados
   - Identificar code owners

2. **Primera Semana**:

   - Arreglar un bug pequeño
   - Agregar un test
   - Revisar 3 PRs
   - Documentar un proceso poco claro

3. **Primer Mes**:
   - Entender flujo de datos
   - Conocer cuellos de botella de rendimiento
   - Contribuir feature significativa
   - Mentorar a alguien más nuevo

### Estándares de Comunicación

- **Template de Descripción PR/MR**:

  ```markdown
  ## Qué

  Descripción breve de cambios

  ## Por qué

  Link a issue/requerimiento

  ## Cómo

  Enfoque técnico tomado

  ## Testing

  Cómo verificar cambios

  ## Screenshots

  Si hay cambios de UI

  ## Impacto en Rendimiento

  Benchmarks si es relevante

  ## Plan de Rollback

  Cómo deshacer si surgen problemas
  ```

- **Formato de Mensaje de Commit**:

  ```
  tipo(alcance): asunto (máx 50 caracteres)

  Cuerpo explicando por qué, no qué (cortar en 72 caracteres)

  Fixes #123
  ```

  Tipos: feat, fix, docs, style, refactor, test, chore

### Directrices de Code Review

**Como Revisor**:

- Respuesta en 4 horas durante horario laboral
- Aprobar/Solicitar cambios en 24 horas
- Enfocarse en corrección > estilo
- Sugerir, no exigir
- Proveer ejemplos para feedback complejo
- Revisar problemas de seguridad primero

**Como Autor**:

- Auto-revisar primero
- Mantener PRs bajo 400 líneas
- Responder a feedback en 24 horas
- No tomar feedback personal
- Actualizar según feedback o explicar por qué no

## Puertas de Calidad

### Definición de Terminado

- [ ] Criterios de aceptación cumplidos
- [ ] Tests escritos y pasando (unit + integración)
- [ ] Cobertura de código mantenida o mejorada
- [ ] Documentación actualizada (código + usuario)
- [ ] Benchmarks de rendimiento pasan
- [ ] Escaneo de seguridad pasa
- [ ] Estándares de accesibilidad cumplidos (si es UI)
- [ ] Feature flag configurado (si aplica)
- [ ] Monitoreo/alertas configuradas
- [ ] Revisado por 2+ miembros del equipo
- [ ] Sin TODOs sin resolver sin números de issue
- [ ] Changelog actualizado

### Estrategia de Testing

```python
# Convención de nomenclatura de tests
def test_deberia_[esperado]_cuando_[condicion]():
    # Preparar
    # Actuar
    # Afirmar
```

- **Pirámide de Tests**:

  - 70% Tests unitarios (rápidos, aislados)
  - 20% Tests de integración (API, base de datos)
  - 10% Tests E2E (solo rutas críticas)

- **Directrices de Testing**:
  - Testear comportamiento, no implementación
  - Una aserción por test cuando sea posible
  - Usar builders/factories para datos de test
  - Tests deben ser determinísticos
  - Sin lógica condicional en tests
  - Mockear dependencias externas
  - Testear casos extremos y rutas de error

### Checklist de Seguridad

- [ ] Validación de entrada en todos los datos de usuario
- [ ] Prevención de inyección SQL (queries parametrizadas)
- [ ] Prevención XSS (escapar salida)
- [ ] Verificaciones de autenticación/autorización
- [ ] Datos sensibles encriptados en reposo
- [ ] Secretos en variables de entorno
- [ ] Dependencias escaneadas por vulnerabilidades
- [ ] Rate limiting implementado
- [ ] CORS configurado apropiadamente
- [ ] Headers de seguridad establecidos

## Monitoreo y Observabilidad

### Telemetría Requerida

```javascript
// Ejemplo de logging estructurado
logger.info("Operación completada", {
  operacion: "registro_usuario",
  duracion_ms: 145,
  id_usuario: user.id,
  id_traza: context.traceId,
  ambiente: process.env.NODE_ENV,
});
```

- **Logs**: Estructurados, con IDs de traza
- **Métricas**: KPIs de negocio + técnicos
- **Trazas**: Tracing distribuido para requests
- **Eventos**: Acciones de usuario y eventos del sistema

### Filosofía de Alertas

- Alertar sobre síntomas, no causas
- Cada alerta debe ser accionable
- Incluir link a runbook en alerta
- No más de 5 alertas críticas por servicio
- La fatiga de alertas es un bug

## Respuesta a Incidentes

### Cuando las Cosas se Rompen

1. **Reconocer** - Reclamar incidente en canal
2. **Evaluar** - Severidad (P0-P4) y radio de impacto
3. **Comunicar** - Página de estado + stakeholders
4. **Mitigar** - Detener sangrado, no arreglar causa raíz aún
5. **Resolver** - Verificar operaciones normales
6. **Aprender** - Postmortem sin culpas en 48 horas

### Template de Postmortem

```markdown
## Resumen del Incidente

- Duración:
- Impacto:
- Causa Raíz:

## Línea de Tiempo

[Minuto a minuto durante el incidente]

## Qué Salió Bien

[Cosas que ayudaron]

## Qué Salió Mal

[Factores contribuyentes]

## Elementos de Acción

[Mejoras específicas con dueños y fechas]

## Lecciones Aprendidas

[Conocimiento para compartir con el equipo]
```

## Mejora Continua

### Rituales Semanales del Equipo

- **Lunes**: Planificación y establecimiento de objetivos
- **Miércoles**: Revisión de deuda técnica
- **Viernes**: Hora de aprendizaje (demos, artículos, cursos)

### Prácticas Mensuales

- Actualizaciones de dependencias
- Revisión de rendimiento
- Auditoría de seguridad
- Día de documentación
- Ejercicio de chaos engineering

### Métricas a Rastrear

- Tiempo de entrega para cambios
- Frecuencia de deployment
- Tiempo medio de recuperación (MTTR)
- Tasa de falla de cambios
- Tiempo de respuesta de code review
- Tendencia de cobertura de tests
- Ratio de deuda técnica

## Recordatorios Importantes

### NUNCA

- ❌ Usar `--force` o `--no-verify` sin acuerdo del equipo
- ❌ Deshabilitar tests en lugar de arreglarlos
- ❌ Commitear código que no compila
- ❌ Almacenar secretos en código
- ❌ Ignorar advertencias de seguridad
- ❌ Deployar viernes después de las 3 PM
- ❌ Hacer suposiciones - verificar con datos
- ❌ Optimizar sin perfilar primero
- ❌ Copiar-pegar sin entender
- ❌ Saltar code review, incluso para fixes "urgentes"

### SIEMPRE

- ✅ Commitear código funcional incrementalmente
- ✅ Escribir tests para bugs antes de arreglar
- ✅ Actualizar documentación mientras avanzas
- ✅ Aprender de implementaciones existentes
- ✅ Pedir ayuda después de 3 intentos fallidos
- ✅ Considerar al próximo desarrollador (podrías ser tú)
- ✅ Dejar el código mejor de lo que lo encontraste
- ✅ Celebrar victorias, aprender de fallas
- ✅ Compartir conocimiento proactivamente
- ✅ Tomar descansos y mantener balance vida-trabajo

## Recursos

### Lectura Requerida

- [The Pragmatic Programmer](https://pragprog.com/titles/tpp20/)
- [Clean Code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Site Reliability Engineering](https://sre.google/books/)
- [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201)

### Herramientas y Referencias

- [12 Factor App](https://12factor.net/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Versionado Semántico](https://semver.org/lang/es/)
- [Commits Convencionales](https://www.conventionalcommits.org/es/)
- [Registros de Decisiones de Arquitectura](https://adr.github.io/)

---

_"Hazlo funcionar, hazlo correcto, hazlo rápido" - Kent Beck_

_Última Actualización: [08-08-2025]_
_Versión: 1.0.0_
