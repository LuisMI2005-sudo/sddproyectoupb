# Constitución de RachaEstudio
1. **Stack simple:** Python 3.12+ y biblioteca estándar; pytest solo para pruebas.
2. **Spec primero:** todo cambio funcional debe estar definido en una spec antes de implementarse.
3. **Separación:** core.py contiene lógica pura; cli.py gestiona argumentos y mensajes; storage.py accede a archivos.
4. **Tests:** cada regla funcional debe tener pruebas; ejecutar pytest -q y exigir que pasen antes de darla por terminada.
5. **Persistencia:** usar JSON local, escrituras atómicas y validación; nunca sobrescribir datos corruptos ni cambiar el esquema sin actualizar la spec.
6. **Idioma:** identificadores en inglés y mensajes al usuario en español; toda función pública debe incluir tipos.
