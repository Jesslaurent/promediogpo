#Equipo 4 Gestor de Alumnos
#Crear un sistema con grupo, alumnos y calificaciones
class Alumno:
    def __init__(self, matricula, nombre, apellido_p, apellido_m, semestre):
        self.matricula = matricula
        self.nombre = f"{nombre} {apellido_p} {apellido_m}"
        self.semestre = semestre
        self.calificaciones = [0, 0, 0, 0, 0]  # 3 examenes, act, pia

    def calcular_promedio(self):
        ponderacion = [0.15, 0.15, 0.15, 0.10, 0.35]
        promedio = sum(p * c for p, c in zip(ponderacion, self.calificaciones))
        return promedio

class Grupo:
    def __init__(self, numero_grupo, aula, materia, max_alumnos):
        self.numero_grupo = numero_grupo
        self.aula = aula
        self.materia = materia
        self.max_alumnos = max_alumnos
        self.alumnos = []

    def registrar_alumno(self, alumno):
        if len(self.alumnos) < self.max_alumnos:
            self.alumnos.append(alumno)
            return True
        else:
            return False

    def calcular_promedio_grupo(self):
        promedios = [alumno.calcular_promedio() for alumno in self.alumnos]
        promedio_grupo = sum(promedios) / len(promedios) if len(promedios) > 0 else 0
        return promedio_grupo

def guardar_datos_en_archivo():
    with open("Exposicion.txt", "w") as file:
        anchos_columna = [30, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10]

        encabezados = ["Nombre del alumno", "Matrícula", "Aula", "Semestre", "Grupo", "Examen 1", "Examen 2", "Examen 3", "Actividades", "PIA", "Promedio"]
        encabezados_formateados = [encabezado.ljust(ancho) for encabezado, ancho in zip(encabezados, anchos_columna)]
        file.write("".join(encabezados_formateados) + "\n")

        for grupo in grupos:
            for alumno in grupo.alumnos:
                datos = [
                    alumno.nombre, alumno.matricula, grupo.aula, str(alumno.semestre), str(grupo.numero_grupo),
                    "{:.2f}".format(alumno.calificaciones[0]),  # Examen 1
                    "{:.2f}".format(alumno.calificaciones[1]),  # Examen 2
                    "{:.2f}".format(alumno.calificaciones[2]),  # Examen 3
                    "{:.2f}".format(alumno.calificaciones[3]),  # Actividades
                    "{:.2f}".format(alumno.calificaciones[4]),  # PIA
                    "{:.2f}".format(alumno.calcular_promedio())  # Promedio
                ]
                datos_formateados = [dato.ljust(ancho) for dato, ancho in zip(datos, anchos_columna)]
                file.write("".join(datos_formateados) + "\n")

        file.write("\nPromedio de Grupos:\n")
        for grupo in grupos:
            promedio_grupo = grupo.calcular_promedio_grupo()
            promedio_grupo_str = "{:.2f}".format(promedio_grupo)
            fila_promedio = [grupo.numero_grupo, grupo.materia, promedio_grupo_str]
            fila_promedio_formateada = [dato.ljust(ancho) for dato, ancho in zip(fila_promedio, [10, 10, 30, 10])]
            file.write("".join(fila_promedio_formateada) + "\n")
grupos = []

try:
    with open("Exposicion.txt", "r") as file:
        grupo = None
        for line in file:
            parts = line.strip().split(",")
            if parts[0] == "Grupo":
                numero_grupo, aula, materia, max_alumnos = parts[1:]
                grupo = Grupo(numero_grupo, aula, materia, int(max_alumnos))
                grupos.append(grupo)
            elif parts[0] == "Alumno" and grupo is not None:
                matricula, nombre, apellido_p, apellido_m, semestre = parts[1:6]
                alumno = Alumno(matricula, nombre, apellido_p, apellido_m, semestre)
                calificaciones = [float(calif) for calif in parts[6:]]
                alumno.calificaciones = calificaciones
                grupo.registrar_alumno(alumno)
except FileNotFoundError:
    pass


while True:
    print("\nMenú principal:")
    print("1. Registrar nuevo Grupo")
    print("2. Registrar nuevo alumno")
    print("3. Registrar calificaciones")
    print("4. Consultar promedio final por matrícula de Alumno")
    print("5. Consultar promedio de Grupo por No. Grupo")
    print("6. Consultar Grupo con mejor promedio")
    print("7. Buscar Alumno por nombre o apellidos")
    print("8. Salir")
    opcion = input("Seleccione una opción: ")

    if opcion == "1":
        numero_grupo = input("No. Grupo: ")
        aula = input("Aula: ")
        materia = input("Materia: ")
        max_alumnos = int(input("Cantidad de Alumnos: "))
        grupo = Grupo(numero_grupo, aula, materia, max_alumnos)
        grupos.append(grupo)
        print("Grupo registrado con éxito.")

    elif opcion == "2":
        matricula = input("Matrícula: ")
        nombre = input("Nombre: ")
        apellido_p = input("Apellido Paterno: ")
        apellido_m = input("Apellido Materno: ")
        semestre = input("Semestre: ")
        numero_grupo = input("Número de Grupo: ")
        # Verificar si el grupo existe
        grupo_encontrado = next((grupo for grupo in grupos if grupo.numero_grupo == numero_grupo), None)
        if grupo_encontrado:
            alumno = Alumno(matricula, nombre, apellido_p, apellido_m, semestre)
            if grupo_encontrado.registrar_alumno(alumno):
                print("Alumno registrado en el grupo con éxito.")
                guardar_datos_en_archivo() 
            else:
                print("No hay espacio en el grupo seleccionado para este alumno.")
        else:
            print("Grupo no encontrado. Asegúrate de registrar el grupo primero.")

    elif opcion == "3":
        matricula = input("Matrícula del alumno: ")
        alumno_encontrado = False

        for grupo in grupos:
            for alumno in grupo.alumnos:
                if alumno.matricula == matricula:
                    print(f"Calificaciones del alumno {alumno.nombre}:")
                    examen1 = float(input("Examen 1: "))
                    examen2 = float(input("Examen 2: "))
                    examen3 = float(input("Examen 3: "))
                    actividades = float(input("Actividades: "))
                    pia = float(input("PIA: "))
                    alumno.calificaciones = [examen1, examen2, examen3, actividades, pia]
                    alumno_encontrado = True
                    break

        if not alumno_encontrado:
            print("Matrícula de alumno no encontrada.")

    elif opcion == "4":
        matricula = input("Matrícula del alumno: ")
        alumno_encontrado = False

        for grupo in grupos:
            for alumno in grupo.alumnos:
                if alumno.matricula == matricula:
                    print(f"Nombre del alumno: {alumno.nombre}")
                    promedio = alumno.calcular_promedio()
                    print(f"Promedio: {promedio}")
                    alumno_encontrado = True
                    break

        if not alumno_encontrado:
            print("Matrícula de alumno no encontrada.")

    elif opcion == "5":
        numero_grupo = input("No. Grupo: ")
        grupo_encontrado = False

        for grupo in grupos:
            if grupo.numero_grupo == numero_grupo:
                promedio_grupo = grupo.calcular_promedio_grupo()
                print(f"Promedio del grupo {grupo.numero_grupo}: {promedio_grupo}")
                grupo_encontrado = True
                break

        if not grupo_encontrado:
            print("No se encontró el grupo.")

    elif opcion == "6":
        grupo_mejor_promedio = None
        mejor_promedio = 0

        for grupo in grupos:
            promedio_grupo = grupo.calcular_promedio_grupo()
            if promedio_grupo > mejor_promedio:
                grupo_mejor_promedio = grupo
                mejor_promedio = promedio_grupo

        if grupo_mejor_promedio is not None:
            print(f"El grupo con mejor promedio es el grupo {grupo_mejor_promedio.numero_grupo} con un promedio de {mejor_promedio}.")
        else:
            print("No hay grupos registrados.")

    elif opcion == "7":
        busqueda = input("Introduce el nombre o apellidos del alumno a buscar: ")
        encontrado = False

        for grupo in grupos:
            for alumno in grupo.alumnos:
                if busqueda in alumno.nombre or busqueda in alumno.matricula:
                    print(f"Alumno: {alumno.nombre}, Matrícula: {alumno.matricula}")
                    print(f"Semestre: {alumno.semestre}")
                    print(f"Promedio: {alumno.calcular_promedio()}")
                    encontrado = True

        if not encontrado:
            print("No se encontraron coincidencias.")

    elif opcion == "8":
        print("Saliendo del programa.")
        guardar_datos_en_archivo()
        break

    else:
        print("Opción no válida. Introduce una opción válida del menú.")

    guardar_datos_en_archivo()
