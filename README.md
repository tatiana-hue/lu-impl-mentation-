# lu-impl-mentation-
#include <iostream>
#include <vector>
using namespace std;

// ---------------------
// Node for doubly-linked matrix
// ---------------------
struct Node {
    double value;
    Node* right;
    Node* left;
    Node* up;
    Node* down;

    Node(double v = 0) : value(v), right(nullptr), left(nullptr), up(nullptr), down(nullptr) {}
};

// ---------------------
// Matrix implemented with 2D doubly linked list
// ---------------------
class LinkedMatrix {
private:
    int n;
    Node* head;

public:
    LinkedMatrix(int size) : n(size) {
        head = nullptr;
        build();
    }

    // Build an n x n matrix
    void build() {
        vector<vector<Node*>> nodes(n, vector<Node*>(n));

        // Create all nodes
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                nodes[i][j] = new Node(0);

        // Link them (right, left, up, down)
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (j + 1 < n) nodes[i][j]->right = nodes[i][j + 1];
                if (j - 1 >= 0) nodes[i][j]->left  = nodes[i][j - 1];
                if (i + 1 < n) nodes[i][j]->down  = nodes[i + 1][j];
                if (i - 1 >= 0) nodes[i][j]->up    = nodes[i - 1][j];
            }
        }

        head = nodes[0][0];
    }

    Node* get(int row, int col) {
        Node* p = head;
        for (int i = 0; i < row; i++) p = p->down;
        for (int j = 0; j < col; j++) p = p->right;
        return p;
    }

    void setValue(int row, int col, double val) {
        get(row, col)->value = val;
    }

    double getValue(int row, int col) {
        return get(row, col)->value;
    }

    void print() {
        Node* row = head;
        while (row != nullptr) {
            Node* col = row;
            while (col != nullptr) {
                cout << col->value << "\t";
                col = col->right;
            }
            cout << endl;
            row = row->down;
        }
        cout << endl;
    }

    int size() { return n; }
};

// ---------------------
// LU Decomposition using LinkedMatrix
// ---------------------
void LU_decomposition(LinkedMatrix& A, LinkedMatrix& L, LinkedMatrix& U) {
    int n = A.size();

    for (int i = 0; i < n; i++) {

        // Compute U[i][k]
        for (int k = i; k < n; k++) {
            double sum = 0;
            for (int j = 0; j < i; j++)
                sum += L.getValue(i, j) * U.getValue(j, k);

            double value = A.getValue(i, k) - sum;
            U.setValue(i, k, value);
        }

        // Compute L[k][i]
        for (int k = i; k < n; k++) {
            if (i == k)
                L.setValue(i, i, 1); // Diagonal = 1
            else {
                double sum = 0;
                for (int j = 0; j < i; j++)
                    sum += L.getValue(k, j) * U.getValue(j, i);

                double value = (A.getValue(k, i) - sum) / U.getValue(i, i);
                L.setValue(k, i, value);
            }
        }
    }
}

// ---------------------
// Example usage
// ---------------------
int main() {
    int n = 3;
    LinkedMatrix A(n), L(n), U(n);

    // Example matrix
    double input[3][3] = {
        {2, -1, -2},
        {-4, 6, 3},
        {-4, -2, 8}
    };

    // Put values in the linked matrix
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            A.setValue(i, j, input[i][j]);

    cout << "Matrix A:" << endl;
    A.print();

    LU_decomposition(A, L, U);

    cout << "Matrix L:" << endl;
    L.print();

    cout << "Matrix U:" << endl;
    U.print();

    return 0;
}