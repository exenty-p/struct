# Struct
Вариант 15
Задание 1
Определить комбинированный (структурный) тип для представления информации
по горным вершинам, состоящей из названия вершины, ее высоты, стране
местонахождения, типе горы (вулканическая, складчатая, платообразная и др.).
Ввести информацию по 20 вершинам. Вывести среднее значение высот всех 20
вершин. Затем вывести информацию, отсортированную по убыванию высоты
вершины (рационально переставлять все поля структуры разом). Вывести сведения
по странам местонахождения 4-х восьмитысячников. Реализовать функцию
изменения данных горной вершины по ее названию. В отдельный массив поместить
все горные вершины в одной стране (страну вводить с клавиатуры). Реализовать
вывод отфильтрованных данных в виде оберточной функции.
Задание 2
Считать из текстового файла данные числового поля для вашей.
Задание 3
Сделать чтение / запись своей структуры в бинарный файл.
Оформить в виде подпрограмм.
<img width="1096" height="552" alt="изображение" src="https://github.com/user-attachments/assets/aaed3e6b-4401-4120-90fa-f8f2d74bdeac" />
<img width="1106" height="543" alt="изображение" src="https://github.com/user-attachments/assets/ce8f9398-5ba5-4b52-bdb3-f80a80e42a49" />
<img width="552" height="420" alt="изображение" src="https://github.com/user-attachments/assets/9a7892b7-7047-4c04-8a66-153f2a609dde" />
<img width="1086" height="613" alt="изображение" src="https://github.com/user-attachments/assets/2f3e32e3-56da-4b46-ae45-63cc04ebe77e" />




#include <iostream>
#include <string>
#include <limits>
#include <iomanip>
#include <fstream>

using namespace std;



enum MountType {
    volcanic,
    folded,
    plateau,
    other
};

struct CountryInfo {
    string countryName;
    string continent;
};

struct MountainPeak {
    string name;
    double height;
    CountryInfo location;
    MountType type;
};



string MountainTypeString(MountType type) {
    switch (type) {
    case volcanic: return "Вулканическая";
    case folded:   return "Складчатая";
    case plateau:  return "Платообразная";
    case other:    return "Другая";
    default:       return "Неизвестно";
    }
}

MountType stringMountainType(const string& str) {
    if (str == "Вулканическая" || str == "volcanic") return volcanic;
    if (str == "Складчатая" || str == "folded")   return folded;
    if (str == "Платообразная" || str == "plateau")  return plateau;
    return other;
}



void printPeakInfo(const MountainPeak& peak) {
    cout << left
        << setw(20) << peak.name
        << setw(10) << peak.height
        << setw(25) << peak.location.countryName
        << setw(20) << peak.location.continent
        << MountainTypeString(peak.type) << endl;
}

void printHeader() {
    cout << left
        << setw(20) << "Название"
        << setw(10) << "Высота"
        << setw(25) << "Страна"
        << setw(20) << "Континент"
        << "Тип горы" << endl;
    cout << string(80, '-') << endl;
}



void printFilteredPeaks(const MountainPeak peaks[], int size, const string& filterName) {
    cout << filterName << endl;
    if (size == 0) {
        cout << "Вершины не найдены." << endl;
        return;
    }
    printHeader();
    for (int i = 0; i < size; i++) {
        printPeakInfo(peaks[i]);
    }
    cout << "\nНайдено: " << size << " вершин" << endl;
}



void saveTextToFile(MountainPeak peaks[], int size, const string& filename) {
    ofstream fout(filename);
    if (!fout.is_open()) {
        cout << "Ошибка открытия файла для записи: " << filename << endl;
        return;
    }
    for (int i = 0; i < size; i++) {
        fout << peaks[i].name << "\n";
        fout << peaks[i].height << "\n";
        fout << peaks[i].location.countryName << "\n";
        fout << peaks[i].location.continent << "\n";
        fout << MountainTypeString(peaks[i].type) << "\n";
    }
    fout.close();
    cout << "Данные записаны в файл: " << filename << endl;
}

int loadFromTextFile(MountainPeak peaks[], int maxSize, const string& filename) {
    ifstream fin(filename);
    if (!fin.is_open()) {
        cout << "Ошибка открытия файла для чтения: " << filename << endl;
        return 0;
    }
    int counter = 0;
    string str;
    while (counter < maxSize) {
        if (!getline(fin, peaks[counter].name)) break;
        if (peaks[counter].name.empty()) break;

        fin >> peaks[counter].height;
        fin.ignore();

        getline(fin, peaks[counter].location.countryName);
        getline(fin, peaks[counter].location.continent);
        getline(fin, str);
        peaks[counter].type = stringMountainType(str);
        counter++;
    }
    fin.close();
    cout << "Данные файла \"" << filename << "\" считаны. Вершин: " << counter << endl;
    return counter;
}



void loadHeightsFromFile(MountainPeak peaks[], int size, const string& filename) {
    ifstream fin(filename);
    if (!fin.is_open()) {
        cout << "Ошибка открытия файла высот: " << filename << endl;
        return;
    }
    string name;
    double height;
    int updated = 0;
    while (fin >> name >> height) {
        for (int i = 0; i < size; i++) {
            if (peaks[i].name == name) {
                peaks[i].height = height;
                updated++;
                break;
            }
        }
    }
    fin.close();
    cout << "Обновлено высот из файла " << filename << "\": " << updated << endl;
}



struct MountainPeakBin {
    char   name[64];
    double height;
    char   countryName[64];
    char   continent[32];
    int    type;          
};

void saveBinaryToFile(MountainPeak peaks[], int size, const string& filename) {
    ofstream fout(filename, ios::binary);
    if (!fout.is_open()) {
        cout << "Ошибка открытия бинарного файла для записи: " << filename << endl;
        return;
    }
    fout.write(reinterpret_cast<const char*>(&size), sizeof(int));

    for (int i = 0; i < size; i++) {
        MountainPeakBin bin{};
        strncpy_s(bin.name, peaks[i].name.c_str(), 63);
        bin.height = peaks[i].height;
        strncpy_s(bin.countryName, peaks[i].location.countryName.c_str(), 63);
        strncpy_s(bin.continent, peaks[i].location.continent.c_str(), 31);
        bin.type = static_cast<int>(peaks[i].type);

        fout.write(reinterpret_cast<const char*>(&bin), sizeof(MountainPeakBin));
    }
    fout.close();
    cout << "Данные записаны в бинарный файл: " << filename << endl;
}

int loadFromBinaryFile(MountainPeak peaks[], int maxSize, const string& filename) {
    ifstream fin(filename, ios::binary);
    if (!fin.is_open()) {
        cout << "Ошибка открытия бинарного файла для чтения: " << filename << endl;
        return 0;
    }
    int size = 0;
    fin.read(reinterpret_cast<char*>(&size), sizeof(int));
    if (size > maxSize) size = maxSize;

    for (int i = 0; i < size; i++) {
        MountainPeakBin bin{};
        fin.read(reinterpret_cast<char*>(&bin), sizeof(MountainPeakBin));
        peaks[i].name = bin.name;
        peaks[i].height = bin.height;
        peaks[i].location.countryName = bin.countryName;
        peaks[i].location.continent = bin.continent;
        peaks[i].type = static_cast<MountType>(bin.type);
    }
    fin.close();
    cout << "Данные считаны из бинарного файла \"" << filename << "\". Вершин: " << size << endl;
    return size;
}



double AvgHeight(const MountainPeak peaks[], int size) {
    if (size == 0) return 0.0;
    double sum = 0;
    for (int i = 0; i < size; i++) sum += peaks[i].height;
    return sum / size;
}


void HeightSort(MountainPeak peaks[], int size) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (peaks[j].height < peaks[j + 1].height) {
                MountainPeak temp = peaks[j];
                peaks[j] = peaks[j + 1];
                peaks[j + 1] = temp;
            }
        }
    }
}

void printEightThousandCountries(const MountainPeak peaks[], int size) {
    cout << "\n Страны с горами-восьмитысячниками" << endl;
    cout << left << setw(20) << "Название" << setw(10) << "Высота" << "Страна" << endl;
    int counter = 0;
    for (int i = 0; i < size; i++) {
        if (peaks[i].height > 8000) {
            cout << left << setw(20) << peaks[i].name
                << setw(10) << peaks[i].height
                << peaks[i].location.countryName << endl;
            counter++;
        }
    }
    if (counter == 0)
        cout << "Восьмитысячников не найдено." << endl;
    else
        cout << "\nНайдено: " << counter << " восьмитысячников" << endl;
}

bool changePeakData(MountainPeak peaks[], int size,
    const string& name, double newHeight, MountType newType)
{
    for (int i = 0; i < size; i++) {
        if (peaks[i].name == name) {
            peaks[i].height = newHeight;
            peaks[i].type = newType;
            return true;
        }
    }
    return false;
}



int CountryFilter(const MountainPeak peaks[], int size,
    MountainPeak result[], const string& country)
{
    int counter = 0;
    for (int i = 0; i < size; i++) {
        if (peaks[i].location.countryName.find(country) != string::npos) {
            result[counter++] = peaks[i];
        }
    }
    return counter;
}



int PeaksMassive(MountainPeak peaks[]) {
    int i = 0;
    peaks[i++] = { "Эверест",       8848, {"Непал/Китай",     "Азия"},              folded };
    peaks[i++] = { "K2",            8611, {"Пакистан/Китай",  "Азия"},              folded };
    peaks[i++] = { "Канченджанга",  8586, {"Непал/Индия",     "Азия"},              folded };
    peaks[i++] = { "Лхоцзе",        8516, {"Непал/Китай",     "Азия"},              plateau };
    peaks[i++] = { "Макалу",        8485, {"Непал/Китай",     "Азия"},              folded };
    peaks[i++] = { "Чо-Ойю",        8188, {"Непал/Китай",     "Азия"},              folded };
    peaks[i++] = { "Дхаулагири",    8167, {"Непал",           "Азия"},              folded };
    peaks[i++] = { "Манаслу",       8163, {"Непал",           "Азия"},              plateau };
    peaks[i++] = { "Нангапарбат",   8126, {"Пакистан",        "Азия"},              folded };
    peaks[i++] = { "Аннапурна I",   8091, {"Непал",           "Азия"},              folded };
    peaks[i++] = { "Гашербрум I",   8080, {"Пакистан/Китай",  "Азия"},              folded };
    peaks[i++] = { "Броуд-пик",     8051, {"Пакистан/Китай",  "Азия"},              folded };
    peaks[i++] = { "Гашербрум II",  8034, {"Пакистан/Китай",  "Азия"},              folded };
    peaks[i++] = { "Шишапангма",    8027, {"Китай",           "Азия"},              folded };
    peaks[i++] = { "Килиманджаро",  5895, {"Танзания",        "Африка"},            volcanic };
    peaks[i++] = { "Аконкагуа",     6962, {"Аргентина",       "Южная Америка"},     folded };
    peaks[i++] = { "Денали",        6190, {"США",             "Северная Америка"},  other };
    peaks[i++] = { "Эльбрус",       5642, {"Россия",          "Европа"},            volcanic };
    peaks[i++] = { "Монблан",       4809, {"Франция/Италия",  "Европа"},            folded };
    peaks[i++] = { "Фудзи",         3776, {"Япония",          "Азия"},              volcanic };
    return i;
}



void createSampleHeightsFile(const string& filename) {
    ofstream fout(filename);
    if (!fout.is_open()) return;
    // Обновим высоты нескольких вершин
    fout << "Эверест 8850\n";
    fout << "Эльбрус 5643\n";
    fout << "Фудзи   3777\n";
    fout.close();
}



int main() {
    setlocale(LC_ALL, "Russian");

    const int MAX = 20;
    MountainPeak peaks[MAX];
    MountainPeak filteredPeaks[MAX];

    int size = PeaksMassive(peaks);

    cout << "\n Все горные вершины \n";
    printHeader();
    for (int i = 0; i < size; i++) printPeakInfo(peaks[i]);

    cout << "\nСредняя высота всех вершин: "
        << fixed << setprecision(2) << AvgHeight(peaks, size) << " м" << endl;

    HeightSort(peaks, size);
    cout << "\n Вершины, отсортированные по убыванию высоты \n";
    printHeader();
    for (int i = 0; i < size; i++) printPeakInfo(peaks[i]);

    printEightThousandCountries(peaks, size);

    cout << "\n Изменение данных вершины" << endl;
    string changeName;
    cout << "Введите название вершины: ";
    getline(cin, changeName);

    double newHeight;
    cout << "Введите новую высоту: ";
    cin >> newHeight;

    int newTypeInt;
    cout << "Тип горы (0-Вулканическая, 1-Складчатая, 2-Платообразная, 3-Другая): ";
    cin >> newTypeInt;
    cin.ignore(numeric_limits<streamsize>::max(), '\n');

    if (changePeakData(peaks, size, changeName, newHeight,
        static_cast<MountType>(newTypeInt)))
        cout << "Данные вершины \"" << changeName << "\" изменены." << endl;
    else
        cout << "Вершина \"" << changeName << "\" не найдена." << endl;

    cout << "\n Фильтрация по стране" << endl;
    string country;
    cout << "Введите название страны: ";
    getline(cin, country);

    int filteredCount = CountryFilter(peaks, size, filteredPeaks, country);
    printFilteredPeaks(filteredPeaks, filteredCount, "Вершины в стране: " + country);

    cout << "\n Текстовый файл" << endl;
    saveTextToFile(peaks, size, "mountains.txt");

    MountainPeak loadedPeaks[MAX];
    int loadedSize = loadFromTextFile(loadedPeaks, MAX, "mountains.txt");
    printFilteredPeaks(loadedPeaks, loadedSize, "Данные из текстового файла");

    cout << "\n Обновление высот из внешнего файла" << endl;
    createSampleHeightsFile("heights_update.txt");
    loadHeightsFromFile(peaks, size, "heights_update.txt");

    cout << "\n Бинарный файл" << endl;
    saveBinaryToFile(peaks, size, "mountains.bin");

    MountainPeak binPeaks[MAX];
    int binSize = loadFromBinaryFile(binPeaks, MAX, "mountains.bin");
    printFilteredPeaks(binPeaks, binSize, "Данные из бинарного файла");

    return 0;
}
