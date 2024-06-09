# Train Management System
Anggota Kelompok :
|  NRP|Nama Anggota  |
|--|--|
|5027231015|Farida Qurrotu A'yuna|
|5027231038|Dani Wahyu Anak Ary|
|5027231076|Syela Zeruya Tandi Lalong|

Pada royek kali ini, kelompok kami mengimplementasikan sistem manajemen rute kereta api menggunakan struktur data graf di C++. Sistem ini memungkinkan pengguna untuk mengelola kota (node) dan rute kereta api (edge) di antara mereka, serta menyediakan fungsionalitas untuk menemukan rute terpendek dan semua rute yang mungkin antara dua kota.

## Daftar Isi

- [Fitur](#fitur)
- [Kelas](#kelas)
  - [TrainNode](#trainnode)
  - [TrainGraph](#traingraph)
  - [TrainGraphAdjacencyMatrix](#traingraphadjacencymatrix)
  - [TrainRoute](#trainroute)
  - [TrainTransRoute](#traintransroute)
- [Penggunaan](#penggunaan)
- [Pilihan Menu](#pilihan-menu)
- [Contoh](#contoh)
- [Instalasi](#instalasi)

## Fitur

- Menambahkan kota (node) ke dalam graf.
- Menambahkan rute kereta api (edge) antara dua kota.
- Menghapus rute kereta api antara dua kota.
- Menampilkan rute terpendek antara dua kota menggunakan algoritma Dijkstra.
- Menampilkan semua rute yang mungkin antara dua kota menggunakan pencarian mendalam (DFS).
- Menampilkan matriks ketetanggaan yang merepresentasikan graf rute kereta api.

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

#### Metode Publik
- **TrainNode(const string &cityName)**: Konstruktor untuk menginisialisasi nama kota.
- **virtual ~TrainNode()**: Memastikan pembersihan memori yang benar saat objek dihapus. Mengembalikan nama kota.
- **getName() const**: Mengembalikan nama kota yang disimpan dalam atribut name.

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

#### Metode Virtual Murni
- **void addNode(TrainNode *node)**: Menambahkan node (kota) ke dalam graf.
- **void addEdge(int node1Index, int node2Index)**: Menambahkan edge (rute) antara dua node.
- **void display() const**: Menampilkan graf.

### TrainGraphAdjacencyMatrix

Mengimplementasikan TrainGraph menggunakan matriks ketetanggaan.

```cpp
class TrainGraphAdjacencyMatrix : public TrainGraph
{
protected:
    int nodeCount;
    vector<TrainNode *> nodes;
    vector<vector<int>> adjacencyMatrix;

public:
    TrainGraphAdjacencyMatrix(int size) : nodeCount(0)
    {
        adjacencyMatrix.resize(size, vector<int>(size, 0));
    }

    ~TrainGraphAdjacencyMatrix()
    {
        for (TrainNode *node : nodes)
        {
            delete node;
        }
    }

    void addNode(TrainNode *node) override
    {
        nodes.push_back(node);
        nodeCount++;
    }

    void addEdge(int node1Index, int node2Index) override
    {
        if (node1Index < nodeCount && node2Index < nodeCount)
        {
            adjacencyMatrix[node1Index][node2Index] = 1;
            adjacencyMatrix[node2Index][node1Index] = 1;
        }
    }

    void display() const override
    {
        cout << "Adjacency Matrix:" << endl;
        for (int i = 0; i < nodeCount; i++)
        {
            for (int j = 0; j < nodeCount; j++)
            {
                cout << adjacencyMatrix[i][j] << " ";
            }
            cout << endl;
        }
    }
};
```

#### Metode Publik
- **TrainGraphAdjacencyMatrix(int size)**: Konstruktor untuk menginisialisasi graf dengan ukuran tertentu.
- **~TrainGraphAdjacencyMatrix()**: Destruktor untuk membersihkan memori.
- **void addNode(TrainNode *node)**: Menambahkan node (kota) ke dalam graf.
- **void addEdge(int node1Index, int node2Index)**: Menambahkan edge (rute) antara dua node.
- **void display() const**: Menampilkan graf.

#### Atribut Privat
- **int nodeCount**: Jumlah node dalam graf.
- **vector<TrainNode *> nodes**: Daftar node dalam graf.
- **vector<vector<int>> adjacencyMatrix**: Matriks ketetanggaan untuk merepresentasikan graf.

### TrainRoute

Mewarisi TrainGraphAdjacencyMatrix dan menambahkan fungsionalitas untuk mencari rute terpendek dan semua rute yang mungkin.

```cpp
class TrainRoute : public TrainGraphAdjacencyMatrix
{
public:
    TrainRoute(int size) : TrainGraphAdjacencyMatrix(size) {}

    int findNodeIndex(const string &nodeName)
    {
        for (int i = 0; i < nodeCount; i++)
        {
            if (nodes[i]->getName() == nodeName)
                return i;
        }
        return -1; // Not found
    }

    void displayShortestRoute(const string &startLocation, const string &endLocation)
    {
        int startIndex = findNodeIndex(startLocation);
        int endIndex = findNodeIndex(endLocation);

        if (startIndex == -1 || endIndex == -1)
        {
            cout << "Invalid start or end location." << endl;
            return;
        }

        vector<int> distance(nodeCount, INT_MAX);
        vector<bool> visited(nodeCount, false);
        vector<int> parent(nodeCount, -1);

        distance[startIndex] = 0;

        for (int i = 0; i < nodeCount - 1; i++)
        {
            int u = minDistance(distance, visited);
            visited[u] = true;

            for (int v = 0; v < nodeCount; v++)
            {
                if (!visited[v] && adjacencyMatrix[u][v] && distance[u] != INT_MAX && distance[u] + adjacencyMatrix[u][v] < distance[v])
                {
                    distance[v] = distance[u] + adjacencyMatrix[u][v];
                    parent[v] = u;
                }
            }
        }

        printShortestRoute(startIndex, endIndex, parent);
    }

    void displayAllRoutes(const string &startLocation, const string &endLocation)
    {
        int startIndex = findNodeIndex(startLocation);
        int endIndex = findNodeIndex(endLocation);

        if (startIndex == -1 || endIndex == -1)
        {
            cout << "Invalid start or end location." << endl;
            return;
        }

        vector<int> visited(nodeCount, false);
        vector<int> path;

        dfsAllRoutes(startIndex, endIndex, visited, path);
    }

    void deleteRoute(int node1Index, int node2Index)
    {
        if (node1Index < nodeCount && node2Index < nodeCount)
        {
            adjacencyMatrix[node1Index][node2Index] = 0;
            adjacencyMatrix[node2Index][node1Index] = 0;
            printDeletedRoute(node1Index, node2Index);
        }
    }

    void addRoute(int node1Index, int node2Index)
    {
        if (node1Index < nodeCount && node2Index < nodeCount)
        {
            adjacencyMatrix[node1Index][node2Index] = 1;
            adjacencyMatrix[node2Index][node1Index] = 1;
            printAddedRoute(node1Index, node2Index);
        }
    }

    void displayAdjacencyMatrix() const
    {
        display();
    }

private:
    void dfs(int currentNode, int targetNode, vector<int> &visited, vector<int> &path, bool &found)
    {
        visited[currentNode] = true;
        path.push_back(currentNode);

        if (currentNode == targetNode)
        {
            for (int i = 0; i < path.size(); i++)
            {
                if (i != 0)
                    cout << " -> ";
                cout << nodes[path[i]]->getName();
            }
            cout << endl;
            found = true;
        }
        else
        {
            for (int i = 0; i < nodeCount; i++)
            {
                if (adjacencyMatrix[currentNode][i] && !visited[i])
                {
                    dfs(i, targetNode, visited, path, found);
                }
            }
        }

        path.pop_back();
        visited[currentNode] = false;
    }

    int minDistance(const vector<int> &distance, const vector<bool> &visited)
    {
        int min = INT_MAX, min_index;

        for (int v = 0; v < nodeCount; v++)
        {
            if (!visited[v] && distance[v] <= min)
            {
                min = distance[v];
                min_index = v;
            }
        }

        return min_index;
    }

    void printShortestRoute(int start, int end, const vector<int> &parent)
    {
        if (start == end)
        {
            cout << nodes[start]->getName();
        }
        else if

 (parent[end] == -1)
        {
            cout << "No path from " << nodes[start]->getName() << " to " << nodes[end]->getName() << endl;
        }
        else
        {
            printShortestRoute(start, parent[end]);
            cout << " -> " << nodes[end]->getName();
        }
    }

    void dfsAllRoutes(int currentNode, int targetNode, vector<int> &visited, vector<int> &path)
    {
        visited[currentNode] = true;
        path.push_back(currentNode);

        if (currentNode == targetNode)
        {
            for (int i = 0; i < path.size(); i++)
            {
                if (i != 0)
                    cout << " -> ";
                cout << nodes[path[i]]->getName();
            }
            cout << endl;
        }
        else
        {
            for (int i = 0; i < nodeCount; i++)
            {
                if (adjacencyMatrix[currentNode][i] && !visited[i])
                {
                    dfsAllRoutes(i, targetNode, visited, path);
                }
            }
        }

        path.pop_back();
        visited[currentNode] = false;
    }

    void printRouteHeader() const
    {
        cout << "Route: ";
    }

    void printDeletedRoute(int node1Index, int node2Index)
    {
        cout << "Deleted route between " << nodes[node1Index]->getName() << " and " << nodes[node2Index]->getName() << endl;
    }

    void printAddedRoute(int node1Index, int node2Index)
    {
        cout << "Added route between " << nodes[node1Index]->getName() << " and " << nodes[node2Index]->getName() << endl;
    }
};
```

#### Metode Publik
- **TrainRoute(int size)**: Konstruktor untuk menginisialisasi graf dengan ukuran tertentu.
- **int findNodeIndex(const string &nodeName)**: Mencari indeks node berdasarkan nama kota.
- **void displayShortestRoute(const string &startLocation, const string &endLocation)**: Menampilkan rute terpendek antara dua kota.
- **void displayAllRoutes(const string &startLocation, const string &endLocation)**: Menampilkan semua rute yang mungkin antara dua kota.
- **void deleteRoute(int node1Index, int node2Index)**: Menghapus rute antara dua node.
- **void addRoute(int node1Index, int node2Index)**: Menambahkan rute antara dua node.
- **void displayAdjacencyMatrix() const**: Menampilkan matriks ketetanggaan.

#### Metode Privat
- **void dfs(int currentNode, int targetNode, vector<int> &visited, vector<int> &path, bool &found)**: Metode rekursif untuk mencari rute menggunakan pencarian mendalam (DFS).
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
6. Menampilkan Matriks Ketetanggaan
0. Keluar

### Contoh

Berikut adalah contoh penggunaan program:

```cpp
int main()
{
    TrainRoute TrainRoute(20);

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
        cout << "1. Show Shortest Route\n";
        cout << "2. Show All Routes\n";
        cout << "3. Add City\n";
        cout << "4. Add Route\n";
        cout << "5. Delete Route\n";
        cout << "6. Display Adjacency Matrix\n";
        cout << "0. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        string startLocation, endLocation, cityName;
        int node1Index, node2Index;
        switch (choice)
        {
        case 1:
            cout << "Enter start location: ";
            cin >> startLocation;
            cout << "Enter end location: ";
            cin >> endLocation;
            TrainRoute.displayShortestRoute(startLocation, endLocation);
            break;
        case 2:
            cout << "Enter start location: ";
            cin >> startLocation;
            cout << "Enter end location: ";
            cin >> endLocation;
            TrainRoute.displayAllRoutes(startLocation, endLocation);
            break;
        case 3:
            cout << "Enter city name: ";
            cin >> cityName;
            TrainRoute.addNode(new TrainNode(cityName));
            break;
        case 4:
            cout << "Enter index of first city: ";
            cin >> node1Index;
            cout << "Enter index of second city: ";
            cin >> node2Index;
            TrainRoute.addRoute(node1Index, node2Index);
            break;
        case 5:
            cout << "Enter index of first city: ";
            cin >> node1Index;
            cout << "Enter index of second city: ";
            cin >> node2Index;
            TrainRoute.deleteRoute(node1Index, node2Index);
            break;
        case 6:
            TrainRoute.displayAdjacencyMatrix();
            break;
        case 0:
            cout << "Exiting...\n";
            break;
        default:
            cout << "Invalid choice. Please try again.\n";
            break;
        }
    } while (choice != 0);

    return 0;
}
```

