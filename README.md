#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <math.h>

// ========================================================
//                     INPUT VALIDATION
// ========================================================

int isAlphabetic(const char *str) {
    if (strlen(str) == 0) return 0;
    for (int i = 0; str[i] != '\0'; i++)
        if (!isalpha(str[i]) && str[i] != ' ')
            return 0;
    return 1;
}

int isNumeric(const char *str) {
    if (strlen(str) == 0) return 0;
    for (int i = 0; str[i] != '\0'; i++)
        if (!isdigit(str[i])) return 0;
    return 1;
}

int isFloat(const char *str) {
    int dot = 0, digit = 0;
    if (strlen(str) == 0) return 0;

    for (int i = 0; str[i] != '\0'; i++) {
        if (str[i] == '.') {
            if (++dot > 1) return 0;
        }
        else if (!isdigit(str[i])) return 0;
        else digit++;
    }
    return digit > 0;
}

void inputString(char *str, int size) {
    fgets(str, size, stdin);
    str[strcspn(str, "\n")] = '\0';
}

// ========================================================
//                        STRUCT
// ========================================================

typedef struct {
    int id;
    char name[50];
    char department[50];
    float salary;
    int age;
    int yearsOfExperience;
} Employee;

// ========================================================
//                   FUNCTION DECLARATIONS
// ========================================================

char *selectDepartment();
void addEmployee(Employee *e, int *count, int *nextId);
void displayEmployees(Employee *e, int count);
void updateEmployee(Employee *e, int count);
void deleteEmployee(Employee *e, int *count);
void searchEmployee(Employee *e, int count);
void sortEmployees(Employee *e, int count);
void saveEmployees(Employee *e, int count);
void openSavedFile();
int isDuplicate(Employee *e, int count, int currentIndex, char *name, char *dept, int age, float salary);

// ========================================================
//                DUPLICATE CHECK
// ========================================================

int isDuplicate(Employee *e, int count, int currentIndex,
                char *name, char *dept, int age, float salary) {

    for (int i = 0; i < count; i++) {
        if (i == currentIndex) continue;

        if (strcmp(e[i].name, name) == 0 &&
            strcmp(e[i].department, dept) == 0 &&
            e[i].age == age &&
            fabs(e[i].salary - salary) < 0.001)
        {
            return 1;
        }
    }
    return 0;
}

// ========================================================
//                DEPARTMENT SELECTION
// ========================================================

char *selectDepartment() {
    static char dep[50];
    char temp[20];

    while (1) {
        printf("\nSelect Department:\n");
        printf("1. Production Management\n");
        printf("2. Maintenance Manager\n");
        printf("3. Sales and Market\n");
        printf("4. Finance / Accounts\n");
        printf("5. Human Resources (HR)\n");
        printf("Enter choice: ");

        inputString(temp, sizeof(temp));

        if (!isNumeric(temp)) { 
            printf("Digits only!\n"); 
            continue; 
        }

        switch (atoi(temp)) {
            case 1: strcpy(dep, "Production Management"); return dep;
            case 2: strcpy(dep, "Maintenance Manager"); return dep;
            case 3: strcpy(dep, "Sales and Market"); return dep;
            case 4: strcpy(dep, "Finance / Accounts"); return dep;
            case 5: strcpy(dep, "Human Resources (HR)"); return dep;
            default: printf("Invalid choice! Select 1–5.\n");
        }
    }
}

// ========================================================
//                      ADD EMPLOYEE
// ========================================================

void addEmployee(Employee *e, int *count, int *nextId) {

    if (*count >= 100) {
        printf("Employee limit reached!\n");
        return;
    }

    char temp[50], name[50], dept[50];
    float salary;
    int age;

    while (1) {
        printf("Enter Name: ");
        inputString(temp, sizeof(temp));
        if (isAlphabetic(temp)) break;
        printf("Name must contain only alphabets!\n");
    }
    strcpy(name, temp);

    strcpy(dept, selectDepartment());

    while (1) {
        printf("Enter Salary (3-50 LPA): ");
        inputString(temp, sizeof(temp));
        if (!isFloat(temp)) { printf("Invalid number!\n"); continue; }

        salary = atof(temp);
        if (salary >= 3 && salary <= 50) break;

        printf("Salary must be between 3 and 50 LPA.\n");
    }
    salary *= 100000;

    while (1) {
        printf("Enter Age (18-60): ");
        inputString(temp, sizeof(temp));
        if (!isNumeric(temp)) { printf("Digits only!\n"); continue; }

        age = atoi(temp);
        if (age >= 18 && age <= 60) break;

        printf("Invalid age!\n");
    }

    if (isDuplicate(e, *count, -1, name, dept, age, salary)) {
        printf("Duplicate employee found. Add anyway? (y/n): ");
        inputString(temp, sizeof(temp));
        if (!(temp[0] == 'y' || temp[0] == 'Y')) {
            printf("Cancelled.\n");
            return;
        }
    }

    e[*count].id = (*nextId)++;
    strcpy(e[*count].name, name);
    strcpy(e[*count].department, dept);
    e[*count].salary = salary;
    e[*count].age = age;
    e[*count].yearsOfExperience = 0;

    (*count)++;
    printf("Employee added successfully!\n");
}

// ========================================================
//                    DISPLAY EMPLOYEES
// ========================================================

void displayEmployees(Employee *e, int count) {
    if (count == 0) {
        printf("No employees yet!\n");
        return;
    }

    printf("\n----------- EMPLOYEE LIST ------------\n");
    for (int i = 0; i < count; i++) {
        printf("ID:%d | %-15s | %-22s | %.2f LPA | Age:%d | Exp:%d yrs\n",
               e[i].id, e[i].name, e[i].department,
               e[i].salary / 100000, e[i].age, e[i].yearsOfExperience);
    }
}

// ========================================================
//                   UPDATE EMPLOYEE
// ========================================================

void updateEmployee(Employee *e, int count) {
    char temp[50];
    printf("Enter Employee ID to update: ");
    inputString(temp, sizeof(temp));

    if (!isNumeric(temp)) { printf("Invalid!\n"); return; }

    int id = atoi(temp);

    for (int i = 0; i < count; i++) {
        if (e[i].id == id) {

            while (1) {
                printf("\nWhat do you want to update?\n");
                printf("1. Name\n2. Department\n3. Salary\n4. Age\n5. Increment Experience\n");
                printf("Enter choice: ");
                inputString(temp, sizeof(temp));

                if (!isNumeric(temp)) { printf("Digits only!\n"); continue; }

                int choice = atoi(temp);
                char newName[50], newDept[50];
                float newSalary = e[i].salary;
                int newAge = e[i].age;

                if (choice == 1) {
                    printf("Enter new Name: ");
                    inputString(newName, sizeof(newName));
                    strcpy(newDept, e[i].department);
                }
                else if (choice == 2) {
                    strcpy(newDept, selectDepartment());
                    strcpy(newName, e[i].name);
                }
                else if (choice == 3) {
                    while (1) {
                        printf("Enter new Salary (>= %.2f LPA): ", e[i].salary / 100000);
                        inputString(temp, sizeof(temp));
                        if (!isFloat(temp)) { printf("Digits only!\n"); continue; }
                        newSalary = atof(temp) * 100000;
                        if (newSalary >= e[i].salary) break;
                        printf("Cannot decrease salary!\n");
                    }
                    strcpy(newName, e[i].name);
                    strcpy(newDept, e[i].department);
                }
                else if (choice == 4) {
                    while (1) {
                        printf("Enter new Age (18-60): ");
                        inputString(temp, sizeof(temp));
                        if (!isNumeric(temp)) { printf("Digits only!\n"); continue; }
                        newAge = atoi(temp);
                        if (newAge >= 18 && newAge <= 60) break;
                        printf("Invalid age!\n");
                    }
                    strcpy(newName, e[i].name);
                    strcpy(newDept, e[i].department);
                }
                else if (choice == 5) {
                    e[i].yearsOfExperience++;
                    printf("Experience incremented ? %d years\n", e[i].yearsOfExperience);
                }
                else {
                    printf("Invalid option!\n");
                    continue;
                }

                if (choice != 5) {
                    if (isDuplicate(e, count, i, newName, newDept, newAge, newSalary)) {
                        printf("Duplicate found. Update cancelled.\n");
                    } else {
                        strcpy(e[i].name, newName);
                        strcpy(e[i].department, newDept);
                        e[i].age = newAge;
                        e[i].salary = newSalary;
                        printf("Update successful!\n");
                    }
                }

                printf("Update more? (y/n): ");
                inputString(temp, sizeof(temp));
                if (!(temp[0] == 'y' || temp[0] == 'Y')) return;
            }
        }
    }

    printf("Employee ID not found!\n");
}

// ========================================================
//                      DELETE EMPLOYEE
// ========================================================

void deleteEmployee(Employee *e, int *count) {
    char temp[20];
    printf("Enter Employee ID to delete: ");
    inputString(temp, sizeof(temp));

    if (!isNumeric(temp)) { printf("Invalid!\n"); return; }

    int id = atoi(temp);

    for (int i = 0; i < *count; i++) {
        if (e[i].id == id) {
            printf("Delete %s? (y/n): ", e[i].name);
            inputString(temp, sizeof(temp));

            if (temp[0] == 'y' || temp[0] == 'Y') {
                for (int j = i; j < *count - 1; j++)
                    e[j] = e[j + 1];

                (*count)--;
                printf("Deleted.\n");
            } else {
                printf("Cancelled.\n");
            }
            return;
        }
    }
    printf("Employee not found!\n");
}

// ========================================================
//                    SEARCH EMPLOYEE
// ========================================================

void searchEmployee(Employee *e, int count) {
    char temp[20];
    printf("Enter Employee ID: ");
    inputString(temp, sizeof(temp));

    if (!isNumeric(temp)) { printf("Invalid!\n"); return; }

    int id = atoi(temp);

    for (int i = 0; i < count; i++) {
        if (e[i].id == id) {
            printf("\nFOUND: %s | %s | %.2f LPA | Age:%d | Exp:%d yrs\n",
                   e[i].name, e[i].department,
                   e[i].salary / 100000,
                   e[i].age, e[i].yearsOfExperience);
            return;
        }
    }
    printf("Employee not found!\n");
}

// ========================================================
//                        SORT EMPLOYEES
// ========================================================

void sortEmployees(Employee *e, int count) {
    if (count == 0) { printf("No employees to sort!\n"); return; }

    char temp[10];
    printf("Sort by: 1-ID 2-Age 3-Salary 4-Name\nEnter: ");
    inputString(temp, sizeof(temp));

    if (!isNumeric(temp)) { printf("Invalid!\n"); return; }

    int choice = atoi(temp);

    for (int i = 0; i < count - 1; i++) {
        for (int j = i + 1; j < count; j++) {
            int swap = 0;

            if      (choice == 1) swap = (e[i].id > e[j].id);
            else if (choice == 2) swap = (e[i].age > e[j].age);
            else if (choice == 3) swap = (e[i].salary > e[j].salary);
            else if (choice == 4) swap = (strcmp(e[i].name, e[j].name) > 0);
            else { printf("Invalid choice!\n"); return; }

            if (swap) {
                Employee t = e[i];
                e[i] = e[j];
                e[j] = t;
            }
        }
    }

    printf("Sorted!\n");
    displayEmployees(e, count);
}

// ========================================================
//                   SAVE EMPLOYEES
// ========================================================

void saveEmployees(Employee *e, int count) {
    char filename[50];
    printf("Enter filename: ");
    inputString(filename, sizeof(filename));

    FILE *fp = fopen(filename, "w");
    if (!fp) { printf("Cannot save!\n"); return; }

    for (int i = 0; i < count; i++)
        fprintf(fp, "%d|%s|%s|%.2f|%d|%d\n",
                e[i].id, e[i].name, e[i].department,
                e[i].salary / 100000, e[i].age, e[i].yearsOfExperience);

    fclose(fp);
    printf("Saved to %s\n", filename);
}

// ========================================================
//                   OPEN SAVED FILE
// ========================================================

void openSavedFile() {
    char filename[50];
    printf("Enter filename to open: ");
    inputString(filename, sizeof(filename));

    FILE *fp = fopen(filename, "r");
    if (!fp) { printf("Cannot open file!\n"); return; }

    printf("\n--- FILE CONTENTS (%s) ---\n", filename);

    char line[200];
    while (fgets(line, sizeof(line), fp))
        printf("%s", line);

    fclose(fp);
}

// ========================================================
//                           MAIN
// ========================================================

int main() {
    Employee employees[100];
    int count = 0;
    int nextId = 1;

    char temp[10];

    while (1) {
        printf("\n==============================\n");
        printf("      EMPLOYEE SYSTEM\n");
        printf("==============================\n");
        printf("1. Add Employee\n");
        printf("2. Display Employees\n");
        printf("3. Update Employee\n");
        printf("4. Delete Employee\n");
        printf("5. Search Employee\n");
        printf("6. Sort Employees\n");
        printf("7. Save Employees\n");
        printf("8. Open Saved File\n");
        printf("9. Exit\n");
        printf("Enter choice: ");

        inputString(temp, sizeof(temp));

        if (!isNumeric(temp)) { printf("Invalid!\n"); continue; }

        int choice = atoi(temp);

        switch (choice) {
            case 1: addEmployee(employees, &count, &nextId); break;
            case 2: displayEmployees(employees, count); break;
            case 3: updateEmployee(employees, count); break;
            case 4: deleteEmployee(employees, &count); break;
            case 5: searchEmployee(employees, count); break;
            case 6: sortEmployees(employees, count); break;
            case 7: saveEmployees(employees, count); break;
            case 8: openSavedFile(); break;
            case 9: printf("Exiting...\n"); return 0;
            default: printf("Invalid Option!\n");
        }
    }
}

