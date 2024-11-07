# BadCodeMVC2

### Caso de Estudio: Plataforma de Adopción de Plantas Raras

**Descripción**: Esta plataforma permite a los usuarios ver plantas raras disponibles para adopción, adoptar una planta, y registrar información de cuidados y salud de la planta. Las plantas tienen características únicas, como sus requisitos específicos de luz, agua y temperatura. El código contiene problemas en la organización y en la forma en la que se maneja la lógica de negocio y la interacción con la base de datos.

---

### Código con Errores (Bad Code)

A continuación se presenta un código inicial con problemas en la estructura del MVC, lógica de negocio, y prácticas de desarrollo en general.

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Mvc;
using PlantAdoption.Models;
using PlantAdoption.Data;

namespace PlantAdoption.Controllers
{
    public class PlantController : Controller
    {
        private readonly DatabaseContext _db;

        public PlantController()
        {
            _db = new DatabaseContext(); // No se usa Inyección de Dependencias
        }

        public IActionResult Index()
        {
            // Consulta dentro de un bucle, poco optimizado
            var plantsList = new List<Plant>();
            foreach (var plantId in _db.Plants)
            {
                plantsList.Add(_db.Plants.Find(plantId));
            }

            // Pasar modelo completo a la vista
            return View(plantsList);
        }

        public IActionResult Details(int id)
        {
            // Lógica de negocio en el controlador
            var plant = _db.Plants.Find(id);
            if (plant == null)
            {
                throw new Exception("Planta no encontrada"); // Sin manejo de excepciones
            }

            // Sin validación del ID
            return View(plant);
        }

        [HttpPost]
        public IActionResult Adopt(Plant plant)
        {
            // Sin validación del modelo
            if (!ModelState.IsValid)
            {
                return View(plant);
            }

            // Lógica de negocio en el controlador en lugar de un servicio
            plant.AdoptionDate = DateTime.Now;
            _db.Plants.Add(plant);
            _db.SaveChanges();

            return RedirectToAction("Index");
        }

        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                // Liberación incorrecta de recursos
                _db.Dispose();
            }
            base.Dispose(disposing);
        }
    }
}
```

---

### Problemas Presentes en el Código

1. **Lógica de Negocio en el Controlador**: El controlador maneja lógica de negocio relacionada con la adopción de plantas y los cuidados. Debería estar en un servicio separado.
2. **Inyección de Dependencias no Utilizada**: `DatabaseContext` se crea directamente en el controlador, lo que dificulta el testing y aumenta el acoplamiento.
3. **Consultas Ineficientes**: En el método `Index`, se realizan consultas en un bucle en lugar de obtener todos los registros de una vez.
4. **Sin Manejo de Excepciones**: Las excepciones no se manejan adecuadamente, lo que puede provocar caídas en la aplicación.
5. **Sin Validación de Entradas**: No se valida si el ID es válido en el método `Details`.
6. **Uso de Modelos Complejos en la Vista**: El modelo completo `Plant` se pasa directamente a la vista, lo que puede exponer datos sensibles.
7. **Ausencia de DTOs**: No se utilizan Data Transfer Objects (DTOs) para reducir y controlar la información enviada a la vista.
8. **Sin Manejo del Ciclo de Vida de Objetos**: La disposición del `DatabaseContext` es incorrecta y puede provocar problemas de rendimiento y fugas de memoria.
9. **Falta de Separación de Responsabilidades (SRP)**: El controlador realiza múltiples tareas, como manejar excepciones, validar datos y manipular la base de datos.
10. **Código Sin Documentación**: No hay comentarios ni documentación para explicar el funcionamiento de cada parte, dificultando el mantenimiento.

---

### Tabla de Objetivos de Mejora

| **#** | **Objetivo de Mejora**                                                                                 | **Descripción**                                                                                                      |
|-------|--------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| 1     | Separar la lógica de negocio del controlador                                                           | Mover la lógica relacionada con la adopción de plantas a un servicio independiente.                                 |
| 2     | Implementar Inyección de Dependencias (DI)                                                             | Usar DI para `DatabaseContext`, permitiendo pruebas unitarias y reducción de acoplamiento.                          |
| 3     | Optimizar las consultas a la base de datos                                                              | Obtener todos los registros de plantas de una sola vez, en lugar de usar un bucle con `Find`.                       |
| 4     | Manejar excepciones de forma adecuada                                                                   | Agregar bloques `try-catch` y mensajes de error claros en cada método.                                              |
| 5     | Validar entradas de usuario en el modelo y el controlador                                               | Asegurarse de que los datos, como el `id`, sean válidos antes de procesarlos.                                       |
| 6     | Utilizar Data Transfer Objects (DTOs)                                                                   | Implementar DTOs para controlar los datos enviados a la vista y reducir el riesgo de exposición de datos sensibles. |
| 7     | Implementar mapeo automático (AutoMapper)                                                               | Automatizar el mapeo entre modelos y DTOs para evitar código repetitivo y simplificar el código.                    |
| 8     | Mejorar el manejo del ciclo de vida de los objetos                                                      | Dejar que el contenedor de DI gestione la disposición del `DatabaseContext`.                                        |
| 9     | Aplicar principios SOLID                                                                                | Enfocarse en SRP y OCP para una mejor organización del código y facilitar la escalabilidad.                        |
| 10    | Añadir comentarios y documentación del código                                                          | Documentar cada método y clase para facilitar el mantenimiento.                                                     |

---
