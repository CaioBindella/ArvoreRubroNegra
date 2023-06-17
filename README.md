#include <stdio.h>
#include <stdlib.h>

// Definição dos possíveis valores da cor de um nó
typedef enum { RED, BLACK } Color;

// Definição de um nó da árvore
typedef struct Node {
    int key;
    struct Node* parent;
    struct Node* left;
    struct Node* right;
    Color color;
} Node;

// Definição da árvore
typedef struct {
    Node* root;
    Node* nil; // Sentinel
} RedBlackTree;

// Função para criar um novo nó
Node* createNode(int key, Color color, Node* parent, Node* left, Node* right) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->key = key;
    node->color = color;
    node->parent = parent;
    node->left = left;
    node->right = right;
    return node;
}

// Função para criar uma nova árvore rubro-negra
RedBlackTree* createRedBlackTree() {
    RedBlackTree* tree = (RedBlackTree*)malloc(sizeof(RedBlackTree));
    Node* nilNode = createNode(0, BLACK, NULL, NULL, NULL);
    tree->nil = nilNode;
    tree->root = tree->nil;
    return tree;
}

// Função auxiliar para realizar uma rotação à esquerda
void leftRotate(RedBlackTree* tree, Node* x) {
    Node* y = x->right;
    x->right = y->left;

    if (y->left != tree->nil) {
        y->left->parent = x;
    }

    y->parent = x->parent;

    if (x->parent == tree->nil) {
        tree->root = y;
    } else if (x == x->parent->left) {
        x->parent->left = y;
    } else {
        x->parent->right = y;
    }

    y->left = x;
    x->parent = y;
}

// Função auxiliar para realizar uma rotação à direita
void rightRotate(RedBlackTree* tree, Node* y) {
    Node* x = y->left;
    y->left = x->right;

    if (x->right != tree->nil) {
        x->right->parent = y;
    }

    x->parent = y->parent;

    if (y->parent == tree->nil) {
        tree->root = x;
    } else if (y == y->parent->left) {
        y->parent->left = x;
    } else {
        y->parent->right = x;
    }

    x->right = y;
    y->parent = x;
}

// Função auxiliar para manter as propriedades da árvore após a inserção
void insertFixup(RedBlackTree* tree, Node* z) {
    while (z->parent->color == RED) {
        if (z->parent == z->parent->parent->left) {
            Node* y = z->parent->parent->right;

            if (y->color == RED) {
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent->color = RED;
                z = z->parent->parent;
            } else {
                if (z == z->parent->right) {
                    z = z->parent;
                    leftRotate(tree, z);
                }

                z->parent->color = BLACK;
                z->parent->parent->color = RED;
                rightRotate(tree, z->parent->parent);
            }
        } else {
            Node* y = z->parent->parent->left;

            if (y->color == RED) {
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent->color = RED;
                z = z->parent->parent;
            } else {
                if (z == z->parent->left) {
                    z = z->parent;
                    rightRotate(tree, z);
                }

                z->parent->color = BLACK;
                z->parent->parent->color = RED;
                leftRotate(tree, z->parent->parent);
            }
        }
    }

    tree->root->color = BLACK;
}

// Função para inserir um valor na árvore
void insert(RedBlackTree* tree, int key) {
    Node* y = tree->nil;
    Node* x = tree->root;

    while (x != tree->nil) {
        y = x;

        if (key < x->key) {
            x = x->left;
        } else {
            x = x->right;
        }
    }

    Node* z = createNode(key, RED, y, tree->nil, tree->nil);

    if (y == tree->nil) {
        tree->root = z;
    } else if (key < y->key) {
        y->left = z;
    } else {
        y->right = z;
    }

    insertFixup(tree, z);
}

// Função auxiliar para buscar um valor na árvore
Node* search(Node* x, int key) {
    if (x == NULL || x->key == key) {
        return x;
    }

    if (key < x->key) {
        return search(x->left, key);
    } else {
        return search(x->right, key);
    }
}

// Função para buscar um valor na árvore
Node* find(RedBlackTree* tree, int key) {
    return search(tree->root, key);
}

// Função auxiliar para encontrar o nó mínimo da subárvore
Node* findMinimum(Node* x) {
    while (x->left != NULL) {
        x = x->left;
    }
    return x;
}

// Função auxiliar para substituir um nó por outro
void transplant(RedBlackTree* tree, Node* u, Node* v) {
    if (u->parent == tree->nil) {
        tree->root = v;
    } else if (u == u->parent->left) {
        u->parent->left = v;
    } else {
        u->parent->right = v;
    }

    v->parent = u->parent;
}

// Função auxiliar para manter as propriedades da árvore após a exclusão
void deleteFixup(RedBlackTree* tree, Node* x) {
    while (x != tree->root && x->color == BLACK) {
        if (x == x->parent->left) {
            Node* w = x->parent->right;

            if (w->color == RED) {
                w->color = BLACK;
                x->parent->color = RED;
                leftRotate(tree, x->parent);
                w = x->parent->right;
            }

            if (w->left->color == BLACK && w->right->color == BLACK) {
                w->color = RED;
                x = x->parent;
            } else {
                if (w->right->color == BLACK) {
                    w->left->color = BLACK;
                    w->color = RED;
                    rightRotate(tree, w);
                    w = x->parent->right;
                }

                w->color = x->parent->color;
                x->parent->color = BLACK;
                w->right->color = BLACK;
                leftRotate(tree, x->parent);
                x = tree->root;
            }
        } else {
            Node* w = x->parent->left;

            if (w->color == RED) {
                w->color = BLACK;
                x->parent->color = RED;
                rightRotate(tree, x->parent);
                w = x->parent->left;
            }

            if (w->right->color == BLACK && w->left->color == BLACK) {
                w->color = RED;
                x = x->parent;
            } else {
                if (w->left->color == BLACK) {
                    w->right->color = BLACK;
                    w->color = RED;
                    leftRotate(tree, w);
                    w = x->parent->left;
                }

                w->color = x->parent->color;
                x->parent->color = BLACK;
                w->left->color = BLACK;
                rightRotate(tree, x->parent);
                x = tree->root;
            }
        }
    }

    x->color = BLACK;
}

// Função para excluir um valor da árvore
void delete(RedBlackTree* tree, int key) {
    Node* z = find(tree, key);

    if (z == NULL) {
        return; // O valor não foi encontrado na árvore
    }

    Node* y = z;
    Node* x;
    Color yOriginalColor = y->color;

    if (z->left == NULL) {
        x = z->right;
        transplant(tree, z, z->right);
    } else if (z->right == NULL) {
        x = z->left;
        transplant(tree, z, z->left);
    } else {
        y = findMinimum(z->right);
        yOriginalColor = y->color;
        x = y->right;

        if (y->parent == z) {
            x->parent = y;
        } else {
            transplant(tree, y, y->right);
            y->right = z->right;
            y->right->parent = y;
        }

        transplant(tree, z, y);
        y->left = z->left;
        y->left->parent = y;
        y->color = z->color;
    }

    free(z);

    if (yOriginalColor == BLACK) {
        deleteFixup(tree, x);
    }
}

// Função auxiliar para imprimir a árvore em ordem
void inorderTraversal(Node* x) {
    if (x != NULL) {
        inorderTraversal(x->left);
        printf("%d ", x->key);
        inorderTraversal(x->right);
    }
}

// Função para imprimir a árvore em ordem
void printTree(RedBlackTree* tree) {
    inorderTraversal(tree->root);
    printf("\n");
}

int main() {
    RedBlackTree* tree = createRedBlackTree();

    insert(tree, 10);
    insert(tree, 20);
    insert(tree, 30);
    insert(tree, 40);
    insert(tree, 50);

    printf("Árvore em ordem: ");
    printTree(tree);

    int key = 30;
    Node* node = find(tree, key);
    if (node != NULL) {
        printf("Valor %d encontrado na árvore\n", key);
    } else {
        printf("Valor %d não encontrado na árvore\n", key);
    }

    key = 25;
    delete(tree, key);
    printf("Árvore após a exclusão do valor %d: ", key);
    printTree(tree);

    return 0;
}
