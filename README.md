#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Node {
    char month[20]; // Month name
    double power;   // in kW
    double time;    // in hours
    struct Node* left;
    struct Node* right;
} Node;

int cmp(const char* month1, const char* month2) {
    const char* months[] = {"January", "February", "March", "April", "May", "June",
                            "July", "August", "September", "October", "November", "December"};

    int index1 = -1, index2 = -1;
    for (int i = 0; i < 12; ++i) {
        if (strcmp(months[i], month1) == 0) {
            index1 = i;
        }
        if (strcmp(months[i], month2) == 0) {
            index2 = i;
        }
    }

    if (index1 < index2) {
        return -1;
    } else if (index1 > index2) {
        return 1;
    } else {
        return 0;
    }
}

Node* create(const char* month, double power, double time) {
    Node* new_node = (Node*)malloc(sizeof(Node));
    if (new_node != NULL) {
        snprintf(new_node->month, sizeof(new_node->month), "%s", month);
        new_node->power = power;
        new_node->time = time;
        new_node->left = new_node->right = NULL;
    }
    return new_node;
}

Node* insert(Node* root, const char* month, double power, double time) {
    if (root == NULL) {
        return create(month, power, time);
    }

    if (cmp(month, root->month) < 0) {
        root->left = insert(root->left, month, power, time);
    } else if (cmp(month, root->month) > 0) {
        root->right = insert(root->right, month, power, time);
    }

    return root;
}

Node* readFile(Node* root, const char* filename) {
    FILE* file = fopen(filename, "r");
    if (file == NULL) {
        printf("Error opening file: %s\n", filename);
        return root;
    }

    char month[20];
    double power, time;

    while (fscanf(file, "%s %lf %lf", month, &power, &time) == 3) {
        root = insert(root, month, power, time);
    }

    fclose(file);
    return root;
}

double calculateConsumption(Node* root, const char* start_month, const char* end_month) {
    if (root == NULL) {
        return 0.0;
    }

    if (cmp(start_month, end_month) > 0) {
        const char* temp = start_month;
        start_month = end_month;
        end_month = temp;
    }

    double left_sum = calculateConsumption(root->left, start_month, end_month);
    double right_sum = calculateConsumption(root->right, start_month, end_month);

    if (cmp(start_month, root->month) <= 0 && cmp(root->month, end_month) <= 0) {
        // Update the formula for energy calculation
        return (root->power * root->time) + left_sum + right_sum;
    }

    return left_sum + right_sum;
}

void free_tree(Node* root) {
    if (root == NULL) {
        return;
    }

    free_tree(root->left);
    free_tree(root->right);

    free(root);
}

int main() {
    Node* root = NULL;

    const char* filename = "my_chocolate_data.txt";

    root = readFile(root, filename);

    char start_month[20], end_month[20];
    printf("Enter start month: ");
    scanf("%s", start_month);
    printf("Enter end month: ");
    scanf("%s", end_month);

    double total_consumption = calculateConsumption(root, start_month, end_month);

    printf("Total Electricity Consumption for Chocolate Bar Manufacturing (%s to %s): %.2f kWh\n", start_month, end_month, total_consumption);

    free_tree(root);

    return 0;
}
