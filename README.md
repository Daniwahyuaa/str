# Train Management System
Anggota Kelompok :
|  NRP|Nama Anggota  |
|--|--|
|5027231015|Farida Qurrotu A'yuna|
|5027231038|Dani Wahyu Anak Ary|
|5027231076|Syela Zeruya Tandi Lalong|

Pada project kali ini, kelompok kami mengimplementasikan sistem manajemen rute kereta api menggunakan struktur data graf di C++. Sistem ini memungkinkan pengguna untuk mengelola kota (node) dan rute kereta api (edge) di antara mereka, serta menyediakan fungsionalitas untuk menemukan rute terpendek dan semua rute yang mungkin antara dua kota.

## Daftar Isi

- [Fitur](#fitur)
- [Kelas](#kelas)
  - [TrainNode](#trainnode)
  - [TrainGraph](#traingraph)
  - [TrainGraphAdjacencyMatrix](#traingraphadjacencymatrix)
  - [TrainTransRoute](#traintransroute)
  - [TrainRoute](#trainroute)
- [Penggunaan](#penggunaan)
  - [Pilihan Menu](#pilihan-menu)
- [Hasil](#hasil)

## Fitur

- Menambahkan kota (node) ke dalam graf.
- Menambahkan rute kereta api (edge) antara dua kota.
- Menghapus rute kereta api antara dua kota.
- Menampilkan rute terpendek antara dua kota menggunakan algoritma Dijkstra.
- Menampilkan semua rute yang mungkin antara dua kota menggunakan pencarian mendalam (DFS).
- Menampilkan adjacency matrix yang merepresentasikan graf rute kereta api.

## Kelas

### TrainNode

Mewakili sebuah kota dalam graf.

```cpp
class TrainNode
{
public:
    TrainNode(const string &cityName) : name(cityName) {}
    virtual ~TrainNode() {}

    string getName() const
    {
        return name;
    }

protected:
    string name;
};
```

#### Publik
- **TrainNode(const string &cityName)**: Konstruktor untuk menginisialisasi nama kota.
- **virtual ~TrainNode()**: Destruktor untuk memastikan pembersihan memori yang benar saat objek dihapus.
- **getName() const**: Mengembalikan nama kota dari node train.

#### Protected
- **string name**: Menyimpan nama kota yang diwakili oleh node train.

### TrainGraph

Kelas abstrak yang mendefinisikan struktur dari graf kereta api.

```cpp
class TrainGraph
{
public:
    virtual void addNode(TrainNode *node) = 0;
    virtual void addEdge(int node1Index, int node2Index) = 0;
    virtual void display() const = 0;
};
```

#### Publik
- **virtual ~TrainGraph()**: Destruktor untuk memastikan pembersihan memori yang benar saat objek dihapus.
- **void addNode(TrainNode *node)**: Metode virtual murni untuk menambahkan node (kota) ke dalam graf.
- **void addEdge(int node1Index, int node2Index)**: Metode virtual murni untuk menambahkan edge (rute) antara dua node.
- **void display() const**: Metode virtual murni untuk menampilkan graf.

### TrainGraphAdjacencyMatrix

Mengimplementasikan TrainGraph menggunakan matriks ketetanggaan.

```cpp
class TrainGraphAdjacencyMatrix : public TrainGraph
{
public:
    TrainGraphAdjacencyMatrix(int size) : nodeCount(0), adjacencyMatrix(size, vector<int>(size, 0)) {}

    ~TrainGraphAdjacencyMatrix() override
    {
        for (int i = 0; i < nodeCount; i++)
        {
            delete nodes[i];
        }
    }

    void addNode(TrainNode *node) override
    {
        nodes.push_back(node);
        nodeCount++;
    }

    void addEdge(int node1Index, int node2Index) override
    {
        adjacencyMatrix[node1Index][node2Index] = 1;
        adjacencyMatrix[node2Index][node1Index] = 1;
    }

    void display() const override
    {
        cout << "-------------------------------------------------------" << endl;
        cout << "Train Route Graph (Adjacency Matrix):" << endl;

        for (int i = 0; i < nodeCount; i++)
        {
            cout << nodes[i]->getName() << " connected to: ";

            for (int j = 0; j < nodeCount; j++)
            {
                if (adjacencyMatrix[i][j] == 1)
                {
                    cout << nodes[j]->getName() << " - ";
                }
            }
            cout << endl;
        }
        cout << "-------------------------------------------------------" << endl;
    }

protected:
    int nodeCount;
    vector<TrainNode *> nodes;
    vector<vector<int>> adjacencyMatrix;
};
```

#### Publik
- **TrainGraphAdjacencyMatrix(int size)**: Konstruktor untuk Menginisialisasi matriks ketetanggaan berukuran size dengan nilai awal 0 dan menghitung jumlah node awal.
- **~TrainGraphAdjacencyMatrix() override**: Destruktor untuk menghapus semua node yang dialokasikan dalam graf untuk membersihkan memori.
- **void addNode(TrainNode *node)**: Menambahkan node (kota) ke dalam graf.
- **void addEdge(int node1Index, int node2Index)**: Menambahkan edge (rute) antara dua node.
- **void display() const**: Menampilkan graf.

#### Protected
- **int nodeCount**: Menyimpan jumlah node dalam graf.
- **vector<TrainNode *> nodes**: Menyimpan daftar node dalam graf.
- **vector<vector<int>> adjacencyMatrix**: Matriks ketetanggaan untuk merepresentasikan koneksi anat node di dalam graf.

### TrainTransRoute

Merepresentasikan rute transportasi train antar dua lokasi (awal dan akhir)

```cpp
class TrainTransRoute
{
public:
    TrainTransRoute(const string &start, const string &end) : startLocation(start), endLocation(end) {}

private:
    string startLocation;
    string endLocation;
};
```

#### Public
- **TrainTransRoute(const string &start, const string &end)**: Konstruktor untuk menetapkan nilai objek rute dengan lokasi awal dan lokasi akhir yang diberikan.

#### Private
- **string startLocation**: Menyimpan nama kota awal rute train.
- **string endLocation**: Menyimpan nama kota akhir rute train.

### TrainRoute

Mewarisi TrainGraphAdjacencyMatrix dan menambahkan fungsionalitas untuk mencari rute terpendek dan semua rute yang mungkin.

```cpp
class TrainRoute : public TrainGraphAdjacencyMatrix
{
public:
    TrainRoute(int size) : TrainGraphAdjacencyMatrix(size) {}

    int findNodeIndex(const string &nodeName);
    void displayShortestRoute(const string &startLocation, const string &endLocation);
    void displayAllRoutes(const string &startLocation, const string &endLocation);
    void deleteRoute(int node1Index, int node2Index);
    void addRoute(int node1Index, int node2Index);
    void displayAdjacencyMatrix() const;

private:
    void dfs(int currentNode, int targetNode, vector<int> &visited, vector<int> &path, bool &found);
    int minDistance(const vector<int> &distance, const vector<bool> &visited);
    void printShortestRoute(int start, int end, const vector<int> &parent);
    void dfsAllRoutes(int currentNode, int targetNode, vector<int> &visited, vector<int> &path);
    void printRouteHeader() const;
    void printDeletedRoute(int node1Index, int node2Index);
    void printAddedRoute(int node1Index, int node2Index);
};
```

#### Publik
- **TrainRoute(int size)**: Konstruktor untuk menginisialisasi graf dengan ukuran tertentu.
- **int findNodeIndex(const string &nodeName)**: Mencari indeks node berdasarkan nama kota.
- **void displayShortestRoute(const string &startLocation, const string &endLocation)**: Menampilkan rute terpendek antara dua kota.
- **void displayAllRoutes(const string &startLocation, const string &endLocation)**: Menampilkan semua rute yang mungkin dilewati antar dua kota.
- **void deleteRoute(int node1Index, int node2Index)**: Menghapus rute antara dua node.
- **void addRoute(int node1Index, int node2Index)**: Menambahkan rute antara dua node.
- **void displayAdjacencyMatrix() const**: Menampilkan matriks ketetanggaan.

#### Private
- **void dfs(int currentNode, int targetNode, vector<int> &visited, vector<int> &path, bool &found)**: Melakukan pencarian jalur dari node saat ini ke node target menggunakan algoritma mendalam (DFS).
- **int minDistance(const vector<int> &distance, const vector<bool> &visited)**: Mencari node dengan jarak minimum yang belum dikunjungi (digunakan dalam algoritma Dijkstra).
- **void printShortestRoute(int start, int end, const vector<int> &parent)**: Mencetak rute terpendek antara dua node.
- **void dfsAllRoutes(int currentNode, int targetNode, vector<int> &visited, vector<int> &path)**: Metode rekursif untuk mencari semua rute yang mungkin menggunakan DFS.
- **void printRouteHeader() const**: Mencetak header untuk output rute.
- **void printDeletedRoute(int node1Index, int node2Index)**: Mencetak informasi ketika sebuah rute dihapus.
- **void printAddedRoute(int node1Index, int node2Index)**: Mencetak informasi ketika sebuah rute ditambahkan.

## Penggunaan

### Pilihan Menu

Program ini menyediakan antarmuka menu untuk mengelola rute kereta api. Pilihan menu termasuk:

1. Menampilkan Rute Terpendek
2. Menampilkan Semua Rute
3. Menambahkan Kota
4. Menambahkan Rute
5. Menghapus Rute
6. Menampilkan adjacency matrix
0. Keluar

Berikut adalah penggunaan program:

```cpp
int main()
{
    TrainRoute TrainRoute(20);
// menambahkan kota
    TrainRoute.addNode(new TrainNode("Jakarta"));      // 0
    TrainRoute.addNode(new TrainNode("Kediri"));       // 1
    TrainRoute.addNode(new TrainNode("Malang"));       // 2
    TrainRoute.addNode(new TrainNode("Surabaya"));     // 3
    TrainRoute.addNode(new TrainNode("Banyuwangi"));   // 4
    TrainRoute.addNode(new TrainNode("Bandung"));      // 5
    TrainRoute.addNode(new TrainNode("Semarang"));     // 6
    TrainRoute.addNode(new TrainNode("Kutoarjo"));     // 7
    TrainRoute.addNode(new TrainNode("Purwokerto"));   // 8
    TrainRoute.addNode(new TrainNode("Yogyakarta"));   // 9
    TrainRoute.addNode(new TrainNode("Solo"));         // 10
    TrainRoute.addNode(new TrainNode("Nganjuk"));      // 11
    TrainRoute.addNode(new TrainNode("Blitar"));       // 12

    TrainRoute.addEdge(0, 1);   // Jakarta - Kediri
    TrainRoute.addEdge(0, 3);   // Jakarta - Surabaya
    TrainRoute.addEdge(0, 4);   // Jakarta - Banyuwangi
    TrainRoute.addEdge(0, 5);   // Jakarta - Bandung
    TrainRoute.addEdge(0, 6);   // Jakarta - Semarang
    TrainRoute.addEdge(0, 7);   // Jakarta - Kutoarjo
    TrainRoute.addEdge(1, 2);   // Kediri - Malang
    TrainRoute.addEdge(1, 3);   // Kediri - Surabaya
    TrainRoute.addEdge(1, 4);   // Kediri - Banyuwangi
    TrainRoute.addEdge(3, 4);   // Surabaya - Banyuwangi
    TrainRoute.addEdge(6, 7);   // Semarang - Kutoarjo
    TrainRoute.addEdge(7, 8);   // Kutoarjo - Purwokerto
    TrainRoute.addEdge(7, 9);   // Kutoarjo - Yogyakarta
    TrainRoute.addEdge(7, 10);  // Kutoarjo - Solo
    TrainRoute.addEdge(8, 9);   // Purwokerto - Yogyakarta
    TrainRoute.addEdge(10, 11); // Solo - Nganjuk
    TrainRoute.addEdge(10, 12); // Solo - Blitar
    TrainRoute.addEdge(11, 12); // Nganjuk - Blitar
    TrainRoute.display();

    int choice;
    do
    {
        cout << "\nMenu:\n";
        cout << "1. Menunjukkan Rute Terpendek\n";
        cout << "2. Menunjukkan Semua Rute\n";
        cout << "3. Tambah Kota\n";
        cout << "4. Tambah Rute\n";
        cout << "5. Hapus Rute\n";
        cout << "6. Menunjukkan Adjency Matrix\n";
        cout << "0. Exit\n";
        cout << "Pilih Menu: ";
        cin >> choice;

        cin.ignore();

        switch (choice)
        {
        case 1:
        {
            string startLocation, endLocation;
            cout << "Pilih Keberangkatan: ";
            getline(cin, startLocation);

            cout << "Pilih Tujuan: ";
            getline(cin, endLocation);

            TrainRoute.displayShortestRoute(startLocation, endLocation);
            break;
        }
        case 2:
        {
            string startLocation, endLocation;
            cout << "Pilih Keberangkatan: ";
            getline(cin, startLocation);

            cout << "Pilih Tujuan: ";
            getline(cin, endLocation);

            TrainRoute.displayAllRoutes(startLocation, endLocation);
            break;
        }
        case 3:
        {
            string cityName;
            cout << "Masukkan Nama Kota: ";
            getline(cin, cityName);

            TrainRoute.addNode(new TrainNode(cityName));
            cout << "Kota Berhasil Ditambahkan.\n";
            break;
        }
        case 4:
        {
            string city1, city2;
            cout << "Tambahkan 2 Nama Kota yang akan ditambahkan Ke Rute: ";
            cin >> city1 >> city2;

            int node1 = TrainRoute.findNodeIndex(city1);
            int node2 = TrainRoute.findNodeIndex(city2);

            if (node1 != -1 && node2 != -1)
            {
                TrainRoute.addRoute(node1, node2);
            }
            else
            {
                cout << "Kota Tidak Ada. Masukkan kota yang tersedia.\n";
            }
            break;
        }
        case 5:
        {
            string city1, city2;
            cout << "Masukkan 2 Nama Kota yang rutenya akan dihapus: ";
            cin >> city1 >> city2;

            int node1 = TrainRoute.findNodeIndex(city1);
            int node2 = TrainRoute.findNodeIndex(city2);

            if (node1 != -1 && node2 != -1)
            {
                TrainRoute.deleteRoute(node1, node2);
            }
            else
            {
                cout << "Kota Tidak Tersedi. Masukkan Kota Yang Tersedia.\n";
            }
            break;
        }
        case 6:
        {
            TrainRoute.displayAdjencyMatrix();
            break;
        }
        case 0:
            cout << "Keluar...\n";
            break;
        default:
            cout << "Pilihan Tidak Ada. Pilih 0 - 6.\n";
        }
    } while (choice != 0);

    return 0;
}
```

## Hasil
- Tampilan awal program
![image](https://github.com/Daniwahyuaa/str/assets/150106905/8c7bb4f1-3fae-4a9d-a54d-82baaeb27fbf)

- Tampilan program saat memilih menu nomor 1
![image](https://github.com/Daniwahyuaa/str/assets/150106905/b3599f93-d08e-4b80-a6e3-36130b41dd31)

- Tampilan program saat memilih menu nomor 2
![image](https://github.com/Daniwahyuaa/str/assets/150106905/b3599f93-d08e-4b80-a6e3-36130b41dd31)

- Tampilan program saat memilih menu nomor 3
![image](https://github.com/Daniwahyuaa/str/assets/150106905/90b70ee4-a728-4085-8714-23bf4dc6ba55)

- Tampilan program saat memilih menu nomor 4
![image](https://github.com/Daniwahyuaa/str/assets/150106905/2d89abda-1969-40c7-9969-a6a9867357ec)

- Tampilan program saat memilih menu nomor 5
![image](https://github.com/Daniwahyuaa/str/assets/150106905/1b32b0d3-7492-4ef2-bd9f-67fb8380a15b)

- Tampilan program saat memilih menu nomor 6
![image](https://github.com/Daniwahyuaa/str/assets/150106905/514a09c3-f341-42d9-8535-f7aa4ad807d0)



