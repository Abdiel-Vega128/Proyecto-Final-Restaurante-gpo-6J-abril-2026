Esta es una guía estructurada para configurar tu entorno de trabajo bajo la metodología de **Agente Global (.agents)**. Vamos a transformar tu flujo de trabajo en una estructura de "Skills" especializada para automatizar el desarrollo de tu app de veterinaria (basada en el proyecto "proyectorestaurante").

---

## 1. Estructura del Agente Global (.agents)

Primero, definimos el cerebro de tu automatización. Esta estructura permite que el IDE y los scripts de apoyo entiendan qué "Skill" deben aplicar.

### Directorio Raíz: `.agents/`
* **SKILL.md**: Documentación técnica y reglas de los agentes.
* **scripts/**: Automatismos para limpieza de caché, despliegue y generación de modelos.
* **ejemplos/**: Snippets de código UI y lógica Firebase.
* **resources/**: Activos, logos de la veterinaria y esquemas JSON.

---

## 2. Prerrequisitos y Configuración de Entorno

Antes de escribir código, aseguremos que el motor está listo.

### Verificación de Herramientas
Ejecuta esto en tu terminal de **VS Code** o **Antigravity**:
```bash
flutter --version
firebase --version
```

### Instalación de Firebase CLI (Si no existe)
Si el comando `firebase` falla, instala Node.js y luego:
```bash
npm install -g firebase-tools
```

### Conectividad y Proyecto
1.  **Login**: `firebase login` (abre el navegador).
2.  **Configurar FlutterFire**:
    ```bash
    dart pub global activate flutterfire_cli
    flutterfire configure
    ```
    *Selecciona tu proyecto de Firebase Console y marca Android/iOS.*

---

## 3. Configuración del Proyecto Flutter

### Archivo: `pubspec.yaml`
Añade las dependencias necesarias para Firestore y el manejo de estados.

```yaml
name: proyectorestaurante
description: App Veterinaria con CRUD de Empleados.
version: 1.0.0+1

environment:
  sdk: '>=3.0.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.24.0
  cloud_firestore: ^4.14.0
  cupertino_icons: ^1.0.2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0
```

---

## 4. Implementación del CRUD de Empleados

Siguiendo la lógica de **Diseño, Código y Scraping**, organizamos el código en `lib/`.

### Estructura de Archivos Dart
```text
lib/
├── models/
│   └── empleado_model.dart
├── services/
│   └── firebase_service.dart
├── ui/
│   ├── home_screen.dart
│   └── empleado_form.dart
└── main.dart
```

#### A. Modelo de Datos (`lib/models/empleado_model.dart`)
```dart
class Empleado {
  String id;
  String nombre;
  String cargo;

  Empleado({required this.id, required this.nombre, required this.cargo});

  Map<String, dynamic> toMap() => {'nombre': nombre, 'cargo': cargo};

  factory Empleado.fromDoc(String id, Map<String, dynamic> data) =>
      Empleado(id: id, nombre: data['nombre'], cargo: data['cargo']);
}
```

#### B. Servicio Firestore (`lib/services/firebase_service.dart`)

```dart
import 'cloud_firestore.dart';
import '../models/empleado_model.dart';

class FirebaseService {
  final CollectionReference employees = FirebaseFirestore.instance.collection('empleados');

  // CREATE
  Future<void> addEmpleado(String nombre, String cargo) =>
      employees.add({'nombre': nombre, 'cargo': cargo});

  // READ (Stream)
  Stream<List<Empleado>> getEmpleados() {
    return employees.snapshots().map((snap) =>
        snap.docs.map((doc) => Empleado.fromDoc(doc.id, doc.data() as Map<String, dynamic>)).toList());
  }

  // UPDATE
  Future<void> updateEmpleado(String id, String nombre, String cargo) =>
      employees.doc(id).update({'nombre': nombre, 'cargo': cargo});

  // DELETE
  Future<void> deleteEmpleado(String id) => employees.doc(id).delete();
}
```

#### C. Pantalla de Inicio y UI (`lib/ui/home_screen.dart`)
```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/empleado_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Veterinaria - Empleados")),
      body: StreamBuilder<List<Empleado>>(
        stream: _service.getEmpleados(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final empleados = snapshot.data!;
          return ListView.builder(
            itemCount: empleados.length,
            itemBuilder: (context, i) => ListTile(
              title: Text(empleados[i].nombre),
              subtitle: Text(empleados[i].cargo),
              trailing: IconButton(
                icon: Icon(Icons.delete, color: Colors.red),
                onPressed: () => _service.deleteEmpleado(empleados[i].id),
              ),
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showForm(context),
        child: Icon(Icons.add),
      ),
    );
  }

  void _showForm(BuildContext context) {
    // Aquí llamarías a empleado_form.dart para capturar datos
  }
}
```

---

## 5. El Agente en Acción: `SKILL.md`

Para que tu sistema de `.agents` funcione, el archivo de habilidades debe verse así:

```markdown
# Agent Skill: Flutter Veterinary Dev

## Skill: Código
- Generar modelos de datos automáticos a partir de Firebase.
- Inyectar servicios de CRUD en pantallas UI.

## Skill: Diseño
- Seguir paleta de colores: Azul veterinario (#2196F3) y Blanco.
- Widgets responsivos para tablets y móviles.

## Skill: Scraping/Data
- Extraer lista de servicios comunes de veterinarias externas para pre-poblar la DB.
```

> **Siguiente paso:** ¿Deseas que profundicemos en el script de automatización dentro de la carpeta `/scripts` para que el agente genere automáticamente las vistas de edición al detectar un nuevo modelo?
