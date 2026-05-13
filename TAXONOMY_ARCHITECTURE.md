# Arquitectura de Taxonomías: YAML Specs vs. Tablas Relacionales

Este documento explica la decisión arquitectónica de utilizar **Especificaciones YAML (Enfoque GitOps)** en lugar de tablas relacionales estáticas (SQL) para modelar las taxonomías educativas en ColabEdu (Países, Regiones, Estándares, Asignaturas y Niveles).

## 1. Todo el Catálogo vive en Git (Versionado y Trazabilidad)
En un modelo tradicional con tablas en BD, añadir el "Bachillerato Internacional (IB)" requiere crear scripts SQL (`INSERT INTO standards...`) o usar una interfaz de administración. 
Con YAML, el catálogo entero vive en el repositorio `ce-specs`. Si alguien añade un país, abre una Pull Request, ves el Diff exacto de qué asignaturas cambian, quién lo hizo, cuándo, y se puede revertir con un clic. Es infraestructura como código aplicada a la educación.

## 2. Sincronización Total con las Rúbricas y Ejercicios
Nuestras rúbricas y ejercicios ya son YAMLs. Si una rúbrica requiere el tag `std.us.ca.ccss`, tener la definición del estándar también en YAML en el mismo repositorio garantiza **consistencia atómica**. Si despliegas una versión del repositorio `ce-specs`, te aseguras de que las rúbricas y los estándares a los que apuntan viajan siempre juntos. Evita errores de integridad causados por migraciones de base de datos desfasadas.

## 3. Flexibilidad Estructural Extrema (Cero Migraciones)
En un modelo relacional, añadir un campo como `idiomasCooficiales` o `iconoBandera` a un país requeriría un `ALTER TABLE`, una migración de Flyway/Liquibase, y actualizar el código Java.
Con YAML, el motor ya lo guarda todo en un campo flexible `bodyJson` dentro de la tabla genérica `spec_definitions`. Solo hay que añadir el campo al archivo YAML y el código ya puede acceder a él. 

## 4. Evita la Complejidad de las Relaciones (JOINs)
Las jerarquías educativas son caóticas. A veces un estándar aplica a un país entero (IB), a veces a una región (TEKS), a veces es una mezcla. Modelar eso con tablas relacionales requiere complejas tablas puente (`standard_region_mapping`, `level_subject_mapping`).
En un YAML, la estructura anidada o las referencias por `id` son naturales, legibles para un humano y se parsean en milisegundos en memoria, eliminando consultas costosas.

## 5. Optimizado para Inteligencia Artificial (LLMs)
Esta es una ventaja crítica para nuestro modelo de negocio basado en Agentes IA. Si un Agente (como Gemini) necesita entender el contexto educativo de un estudiante para evaluar un ejercicio, pasarle el contenido en texto plano de un YAML bien estructurado es instantáneo y semánticamente perfecto. Intentar que un LLM entienda un volcado de múltiples tablas SQL cruzadas es propenso a errores y costoso en tokens.

---
*Conclusión:* Usamos la base de datos como un motor de búsqueda y caché (ingestando los YAMLs en `spec_definitions`), pero la **"verdad absoluta" (Source of Truth)** es siempre el repositorio de texto `ce-specs`.
