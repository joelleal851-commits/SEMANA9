// Tarea Semana 9 - Manipulacion de archivos CSV en C++17
// Uso: ./inventario [ruta_csv]   (por defecto datos/productos.csv)
#include <fstream>
#include <iomanip>
#include <iostream>
#include <sstream>
#include <string>
#include <vector>
using namespace std;

struct Producto { string codigo, nombre; double precio; int existencia; };

// Separa y valida una fila. Devuelve "" si es valida o el motivo del rechazo.
string validar(const string& linea, Producto& p) {
    stringstream ss(linea);
    string cod, nom, pre, exi, extra;
    if (!(getline(ss, cod, ',') && getline(ss, nom, ',') &&
          getline(ss, pre, ',') && getline(ss, exi, ',')) || getline(ss, extra, ','))
        return "no tiene exactamente 4 campos";
    if (cod.empty()) return "codigo vacio";
    if (nom.empty()) return "nombre vacio";
    try {
        size_t pos;  // pos debe llegar al final: todo el texto debe ser numero
        p.precio = stod(pre, &pos);
        if (pos != pre.size() || !(p.precio > 0)) return "precio invalido";
        p.existencia = stoi(exi, &pos);
        if (pos != exi.size() || p.existencia < 0) return "existencia invalida";
    } catch (...) {
        return "precio o existencia no numericos";
    }
    p.codigo = cod;
    p.nombre = nom;
    return "";
}

int main(int argc, char* argv[]) {
    string ruta = (argc > 1) ? argv[1] : "datos/productos.csv";
    ifstream archivo(ruta);  // solo lectura: el CSV no se modifica
    if (!archivo.is_open()) {
        cerr << "ERROR: no se pudo abrir el archivo '" << ruta << "'" << endl;
        return 1;
    }

    string linea;
    getline(archivo, linea);  // omitir encabezado
    vector<Producto> validos;
    int invalidos = 0, numLinea = 1;
    double total = 0;

    while (getline(archivo, linea)) {
        numLinea++;
        if (!linea.empty() && linea.back() == '\r') linea.pop_back();  // CSV de Windows
        if (linea.empty()) continue;
        Producto p;
        string motivo = validar(linea, p);
        if (motivo.empty()) {
            validos.push_back(p);
            total += p.precio * p.existencia;
        } else {
            invalidos++;
            cout << "Linea " << numLinea << " invalida: " << motivo << endl;
        }
    }
    archivo.close();

    cout << fixed << setprecision(2);
    cout << "Validos: " << validos.size() << " | Invalidos: " << invalidos
         << " | Valor del inventario: Q" << total << endl;

    string buscado, resultado = "El codigo NO existe en el archivo.";
    cout << "Ingrese el codigo a buscar: ";
    getline(cin, buscado);
    for (const auto& p : validos) {
        if (p.codigo == buscado) {
            ostringstream r;
            r << fixed << setprecision(2) << "ENCONTRADO -> " << p.codigo << " | " << p.nombre
              << " | Q" << p.precio << " | existencia: " << p.existencia;
            resultado = r.str();
            break;
        }
    }
    cout << resultado << endl;

    ofstream reporte("reportes/resumen.txt");
    if (!reporte.is_open()) {
        cerr << "ERROR: no se pudo crear reportes/resumen.txt (cree la carpeta reportes)" << endl;
        return 2;
    }
    reporte << fixed << setprecision(2);
    reporte << "Total de registros validos:   " << validos.size() << "\n"
            << "Total de registros invalidos: " << invalidos << "\n"
            << "Valor total del inventario:   Q" << total << "\n"
            << "Resultado de la busqueda:     " << resultado << "\n";
    reporte.close();
    cout << "Reporte generado en reportes/resumen.txt" << endl;
    return 0;
}
