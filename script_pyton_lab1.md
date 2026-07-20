
# TASK MANAGER CLI


## Características

- Autenticación mediante `httpbin.org`.
- Máximo de 3 intentos de acceso.
- Gestión de tareas en memoria.
- Menú interactivo por consola.
- Permite agregar, listar y eliminar tareas.
- Las tareas tienen estado predeterminado `"pendiente"`.

## Código

```python
import requests
import sys
import argparse

tasks = []


def add_task():
    title = input("Ingrese el título de la tarea: ")

    if title.strip() == "":
        print("Error: título vacío.")
        return

    tasks.append({
        "title": title,
        "status": "pendiente"
    })

    print("Tarea agregada.")


def list_tasks():
    if len(tasks) == 0:
        print("No hay tareas.")
        return

    print("\nTareas registradas:")

    for task in tasks:
        print(f"- {task['title']} ({task['status']})")


def delete_task():
    title = input("Título a eliminar: ")

    for task in tasks:
        if task["title"] == title:
            tasks.remove(task)
            print("Tarea eliminada.")
            return

    print("La tarea no existe.")


def authenticate(expected_user, expected_pass):

    for intento in range(3):

        print(f"\nIntento {intento + 1} de 3")

        user = input("Usuario: ")
        passwd = input("Contraseña: ")

        if user == expected_user and passwd == expected_pass:

            try:
                url = f"https://httpbin.org/basic-auth/{user}/{passwd}"

                response = requests.get(
                    url,
                    auth=(user, passwd),
                    timeout=5
                )

                if response.status_code == 200:
                    print("Autenticación correcta.")
                    return True

            except requests.RequestException:
                print("Error de conexión.")

        print("Credenciales incorrectas.")

    return False


def main():

    parser = argparse.ArgumentParser()
    parser.add_argument("-u", "--user", required=True)
    parser.add_argument("-p", "--password", required=True)

    args = parser.parse_args()

    if not authenticate(args.user, args.password):
        print("Acceso denegado.")
        sys.exit()

    while True:

        print("\n--- MENÚ ---")
        print("1. Agregar tarea")
        print("2. Listar tareas")
        print("3. Eliminar tarea")
        print("4. Salir")

        opcion = input("Seleccione una opción: ")

        if opcion == "1":
            add_task()

        elif opcion == "2":
            list_tasks()

        elif opcion == "3":
            delete_task()

        elif opcion == "4":
            print("Hasta luego.")
            break

        else:
            print("Opción inválida.")


if __name__ == "__main__":
    main()
```
