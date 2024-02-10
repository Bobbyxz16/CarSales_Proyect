//LIBRARIES FOR THE PROGRAM
#include <stdio.h>     // Standard Input/Output functions
#include <string.h>    // String manipulation functions
#include <stdlib.h>      // General utilities library
#include <time.h>        // Time-related functions

/* CONSTANTS */
#define DISCOUNT_MEMBER_PERCENTAGE 0.65f    // Discount percentage for members
#define MENU_VIEW_CARS_STOCK 'a'            // Menu option to view available cars in stock
#define MENU_OPTION_BUY_CAR 'b'              // Menu option to buy a car
#define MENU_OPTION_VIEW_SALES 'c'           // Menu option to view sales
#define MENU_OPTION_COSTUMER_FEEDBACK 'd'    // Menu option for customer feedback
#define MENU_OPTION_EXIT 'x'                 // Menu option to exit the program
#define TRUE 1                  // Boolean true value
#define FALSE 0                 // Boolean false value
#define MAX_SALES 10            // Maximum number of sales records
#define TXT_FILE "VIEW_CARS.txt"  // File name for storing view sales information
#define FILE_OPENED 0       // File status: opened successfully
#define FILE_CLOSED 1       // File status: closed successfully
#define FILE_ERROR 2        // File status: encountered an error


/* VARIABLES */
int carAvailable = 100;  // Initial number of available cars
float discountValue;     // Variable of the discount value
unsigned short numberOfSales = 0;  // Counter for the number of sales

unsigned short carAmountPerSale[MAX_SALES];    // Number of cars sold per sale
unsigned short typeOfCarPerSale[MAX_SALES];   // Type of car sold per sale
unsigned short discountGivenPerSale[MAX_SALES];   // Discount given per sale
unsigned short costumerRate[MAX_SALES];    // Customer rating per sale
unsigned short costumerAge[MAX_SALES];    // Customer age per sale
unsigned short carSoldPerType[MAX_SALES];    //Array to store the amount of cars sold per model
char customerNames[MAX_SALES][201];   // Array to store customer names
char costumerFeedback[MAX_SALES][2001];   // Array to store customer feedback
float totalSalesPerCarType[MAX_SALES];    // Array to store total sales per car model

float carPrices[] = {25000.5f, 40000.5f, 67000.7f, 84000.8f, 36500.6f,
                     55800.2f, 72900.3f, 91600.4f, 33390.9f, 48300.4f}; // Array of car prices

char carTypes[][10] = {"FIAT", "SEAT", "FORD", "HYUNDAI", "DACIA",
                       "LEXUS", "MASERATI", "MERCEDES", "NISSAN", "PORSCHE"}; // Array of car types

unsigned short yearOfManufacture[] = {1932, 1898, 1947, 1922, 1843, 1928,
                                      1949, 1878, 1845, 1904}; // Array of manufacturing years

unsigned short remainingCarModel[] = {20, 10, 5, 10, 10,
                                      5, 10, 10, 5, 5}; // Remaining models of each car type

FILE* file;   // File pointer for file operations
unsigned char fileStatus = FILE_CLOSED; // Status of the file (opened, closed, or encountered an error)


/* USER EXPERIENCE */
/*----------------------------------------------------------------------------------------------------------------------*/
//This a way to retrieves a character from the console, but I also validate if the entered character
// is one of the valid characters specified in the validChars string
char getCharFromConsole(char* message, const char *validChars) {
    char userChoice;

    do {
        printf("%s", message);
        scanf(" %c", &userChoice);
        if (strchr(validChars, userChoice) == NULL) {
            printf("Invalid choice. Please enter a valid character.\n"); // Notifying the user of an invalid choice
        }
    } while (strchr(validChars, userChoice) == NULL); // Repeating until a valid character is entered

    return userChoice; // Returning the valid character chosen by the user
}

/*----------------------------------------------------------------------------------------------------------------------*/
// this function give the user a chance to read the previous output, allow them to continue when ready
// customise the output depending on the user's choice
void pauseProgram(char userChoice) {
    if (userChoice == MENU_OPTION_EXIT) {
        printf("\n\nPress Enter to Exit...");
    }
    else {
        printf("\n\nPress Enter to return to the Menu...");
    }
    // two of these to skip the newline character that's likely floating around the console window
    getchar(); getchar();

}

/* FILE ENCRYPTION */
/*----------------------------------------------------------------------------------------------------------------------*/
// Function to perform simple XOR encryption/decryption of the view sales information.
// XOR encryption is very basic and not secure for serious cryptographic purposes.
// For secure encryption, you should use established cryptographic algorithms and libraries like OpenSSL or libsodium.
// Anyway this is a simple example of the file encryption
void XOR(char* inputFileName, char* outputFileName, char* key) {
    FILE* inputFile, *outputFile;
    char byte;
    size_t keyLength = strlen(key);
    size_t i = 0;
    // Opening the input file for reading in binary mode
    inputFile = fopen(inputFileName, "rb");
    if (inputFile == NULL) {
        perror("Error opening input file");
        return;
    }
    // Opening the output file for writing in binary mode
    outputFile = fopen(outputFileName, "wb");
    if (outputFile == NULL) {
        perror("Error opening output file");
        fclose(inputFile);
        return;
    }
    // Performing XOR operation on each byte of the input file using the key
    while (fread(&byte, 1, 1, inputFile) == 1) {
        byte ^= key[i % keyLength]; // XOR operation with the key
        fwrite(&byte, 1, 1, outputFile); // Writing the result to the output file
        i++;
    }
    // Closing the files after processing
    fclose(inputFile);
    fclose(outputFile);
}

/* FILE FUNCTIONS */
/*----------------------------------------------------------------------------------------------------------------------*/
// Function to create a new file with the specified name
FILE* createFile(char fileName[201]) {
    file = fopen(fileName, "w"); // Opening the file in write mode
    if (file != NULL) {
        fclose(file); // Closing the file immediately to create an empty file
    }
    return file; // Returning the file pointer
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to open a file with the specified name and mode
void openFile(char fileName[201], char mode[4]) {
    file = fopen(fileName, mode); // Opening the file with the given mode

    if (file == NULL) { // If the file opening fails
        if (createFile(fileName) == NULL) { // Attempt to create the file
            fileStatus = FILE_ERROR; // Set file status to indicate an error
        } else {
            openFile(fileName, mode); // Recursive call to open the file after successful creation
        }
    } else {
        fileStatus = FILE_OPENED; // Set file status to indicate successful opening
    }
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to close the currently opened file
void closeFile() {
    // only attempt to close the file if it's already open
    if (fileStatus == FILE_OPENED) {
        fclose(file);
        fileStatus = FILE_CLOSED;
    }
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to read data from the opened file and populate arrays
void readDataFromFile() {
    // will keep track of how many lines were read from the file
    int lineCounter = 0;
    char line[2001];
    // this is an infinite loop, we'll manually stop it once we reach the end of the file
    while (1) {
        // Variables to store values read from each line of the file
        unsigned short carAmountPerSaleValue = 0, typeOfCarPerSaleValue = 0, discountGivenPerSaleValue = 0,
                customerAgeValue = 0, costumerRateValue = 0, remainingCarModelValue = 0, carSoldPerTypeValue = 0;
        float totalSalesPerCarTypeValue = 0;
        char customerNameValue[201] = "", costumerFeedbackValue[2001] = "";

        // Read a complete line using fgets
        if (fgets(line, sizeof(line), file) == NULL) {
            break;  // Exit if no more lines can be read
        }

        // Use sscanf to extract values from the line
        int scanResult = sscanf(
                line,
                "%hd,%f,%hd,%hd,%hd,%hd,%hd,%hd,%200[^,],%2000[^\n]%*c",
                &carSoldPerTypeValue, &totalSalesPerCarTypeValue,
                &remainingCarModelValue, &carAmountPerSaleValue, &typeOfCarPerSaleValue,
                &discountGivenPerSaleValue, &customerAgeValue,
                &costumerRateValue, customerNameValue, costumerFeedbackValue
        );

        // if we reached the end of the file
        if (scanResult == EOF) {
            // then, stop the loop
            break;
        }

        // add the bits of data that were read above into the correct arrays
        carAmountPerSale[lineCounter] = carAmountPerSaleValue;
        typeOfCarPerSale[lineCounter] = typeOfCarPerSaleValue;
        costumerAge[lineCounter] = customerAgeValue;
        costumerRate[lineCounter] = costumerRateValue;
        discountGivenPerSale[lineCounter] = discountGivenPerSaleValue;

        // need to use strcpy here because we're working with strings
        strcpy(customerNames[lineCounter], customerNameValue);
        strcpy(costumerFeedback[lineCounter], costumerFeedbackValue);

        // calculate the price of the purchase per car model
        float carTypePrice = carAmountPerSaleValue * carPrices[typeOfCarPerSaleValue];

        // update carAvailable in the file
        carAvailable -= carAmountPerSaleValue;

        // Updated cars have been sold by car model in the file
        carSoldPerType[typeOfCarPerSaleValue] += carAmountPerSaleValue;

        // update the reaming cars per car model in the file
        remainingCarModel[typeOfCarPerSaleValue] -= carAmountPerSaleValue;

        // update total sales per car model in the file
        totalSalesPerCarType[typeOfCarPerSaleValue] += carTypePrice;

        // increment the lineCounter, ready for next line that might be read
        lineCounter++;
    }
    // make sure the numberOfSales variable is also aware of how many sales are available after the above operation
    numberOfSales = lineCounter;
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to get data from a file
void getDataFromFile() {
    openFile(TXT_FILE, "r"); // Opening the file in read mode

    if (fileStatus == FILE_OPENED) {
        readDataFromFile(); // If the file is successfully opened, read data from it
    }
    else if (fileStatus == FILE_ERROR) {
        printf("There was an error trying to read from the file %s.", TXT_FILE);
        //the function is being used to pause the program and give the
        // possibly to allow the user to see the error message before the program terminates.
        pauseProgram('_');
    }

    closeFile();
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to write data to the VIEW_CARS.txt file
void writeDataToFile() {
    for (int i = 0; i < numberOfSales; i++) {
        char line[2001];

        // Format the line using sprintf which is the easiest way to do this
        sprintf(line, "%hd,%f,%hd,%hd,%hd,%hd,%hd,%hd,%s,%s\n",
                carSoldPerType[i], totalSalesPerCarType[i], remainingCarModel[i], carAmountPerSale[i],
                typeOfCarPerSale[i], discountGivenPerSale[i], costumerAge[i], costumerRate[i],
                customerNames[i], costumerFeedback[i]);

        // Write the line to the file using fputs
        fputs(line, file);
    }
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to save data to the VIEW_CARS.txt file
void saveDataToFile() {
    openFile(TXT_FILE, "w");

    if (fileStatus == FILE_OPENED) {
        writeDataToFile();
    }
    else if (fileStatus == FILE_ERROR) {
        printf("There was an error trying to write to the file %s.", TXT_FILE);
        // this function requires a char value, so we give it one
        //     that'll tell it we're using it because of a file error
        //        see the function body, it's been updated to check for
        //        this underscore character
        pauseProgram('_');
    }

    closeFile();
}

/* FROM HERE WE START THE INTERACTIVE CAR SALES PROGRAM WITH THE USER */
/*----------------------------------------------------------------------------------------------------------------------*/
// Function to greet the customer when entering the car sales office
void menu_greetCustomer() {
    printf("\nWelcome to the Car Sales office!\n");
    printf("Is my pleasure for you to choose our office as your choice of purchase\n");
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to display the menu options to the user
void menu_showMenu() {
    // present the various actions the user can choose from
    printf("Menu:\n");
    printf("a. View Cars Stock\n");
    printf("b. Buy Cars\n");
    printf("c. View Sales Data\n");
    printf("d. Customer Feedback\n");
    printf("x. Exit\n\n");

}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to display available car types to the user. It will be used in 'buy cars' function
void menu_showCarTypes() {
    char modelTypes[][10] = {"FIAT", "SEAT", "FORD", "HYUNDAI", "DACIA",
                             "LEXUS", "MASERATI", "MERCEDES", "NISSAN", "PORSCHE"};
    int numberOfCars= sizeof(carPrices) / sizeof(float);
    // show the user the types of Cars
    printf("\nCars Types:\n");
    for (int i = 0; i < numberOfCars; i++) {
        printf("%d - %s - Remaining amount: %d\n", i, modelTypes[i], remainingCarModel[i]);
    }
}
/*----------------------------------------------------------------------------------------------------------------------*/
/* SORTING YEAR OF SALE AMOUNT IN DESCENDING ORDER */
// Function to swap information between prices in the arrays. It will be used the carSortFunction
void swapSales(int i, int j) {
    // Swap carTypes
    char tempType[10];
    strcpy(tempType, carTypes[i]);
    strcpy(carTypes[i], carTypes[j]);
    strcpy(carTypes[j], tempType);

    // Swap carSoldPerType
    unsigned short tempNeeded = carSoldPerType[i];
    carSoldPerType[i] = carSoldPerType[j];
    carSoldPerType[j] = tempNeeded;

    // Swap totalSalesPerCarType
    float tempTotalSales = totalSalesPerCarType[i];
    totalSalesPerCarType[i] = totalSalesPerCarType[j];
    totalSalesPerCarType[j] = tempTotalSales;
}

// Function to sort cars based on sale amount in descending order using the bubble sort algorithm
void caRSorting() {
    // Get the number of elements in the totalSalesPerCarType array
    int n = sizeof(totalSalesPerCarType) / sizeof(totalSalesPerCarType[0]);

    // Iterate through the array using the bubble sort algorithm
    for (int i = 0; i < n - 1; i++) {
        // The outer loop performs a pass through the array
        for (int j = 0; j < n - i - 1; j++) {

            if (totalSalesPerCarType[j] < totalSalesPerCarType[j + 1]) {
                // If the current price is less than the next price, the prices will swap
                swapSales(j, j + 1);
            }
        }
    }
}

void salesDataPerCarModel() {
    caRSorting(); // Sort the cars by sale amount in descending order

    // Show the user the prices of the cars after sorting. It will be used in the view sales function
    int numberOfSalesPerType = sizeof(carPrices) / sizeof(float);
    printf("\n----------------------------------------------\n");
    printf("Data Sales per Car Model\n");
    printf("----------------------------------------------\n");
    for (int i = 0; i < numberOfSalesPerType; i++) {
        printf("%s sold %d units for a total of %f\n", carTypes[i], carSoldPerType[i], totalSalesPerCarType[i]);
    }
    printf("----------------------------------------------\n\n");
}

/*----------------------------------------------------------------------------------------------------------------------*/
/* DISCOUNT FUNCTIONS */

// Function to apply a discount to the current price
float menu_applyDiscount(float currentPrice) {
    return currentPrice * (1 - DISCOUNT_MEMBER_PERCENTAGE);
}
// Function to check if a discount is needed based on membership
unsigned short menu_checkIfDiscountIsNeeded(char memberOfCarSales) {
    unsigned short giveDiscount = FALSE;

    if (memberOfCarSales == 'y') {
        giveDiscount = TRUE;
        discountValue = DISCOUNT_MEMBER_PERCENTAGE;
    }
    return giveDiscount;
}
// Function to print the outcome of applying a discount
void menu_printDiscountOutcome(unsigned short giveDiscount) {
    switch (giveDiscount) {
        case TRUE:
            discountValue *= 100; // Converting discount percentage for display
            printf("\nDiscount of %.0f%% applied!\n", discountValue);
            break;
        case FALSE:
            printf("\nDiscount not applied.\n");
            break;
    }
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function to request and handle customer feedback
// Also putting loops to make sure is using the correct input
void menu_feedbackRequest() {
    unsigned short correctInput ;
    char customerRateInput;

    correctInput = FALSE;

    printf("\nWould you like to provide customer feedback? ('y' or 'n'): \n");

    do {
        scanf(" %c", &customerRateInput);

        if (customerRateInput == 'y') {
            correctInput = TRUE;
            printf("\n\n- CUSTOMER FEEDBACK -");
            printf("\nBefore finishing the purchase, we would love for you to leave your rating (0 - 5): ");
            // Validate and read the customer rating input
            while (TRUE){
                if (scanf("%hd", &costumerRate[numberOfSales]) != 1 ){

                    printf("Incorrect Input. Please type a numeric value: ");
                    while (getchar() != '\n');
                    continue;
                }
                if ( 0 <= costumerRate[numberOfSales] && costumerRate[numberOfSales] <= 5) {
                    break;
                } else {
                    printf("Please enter a rating between 0 and 5: ");
                    while (getchar() != '\n');
                }
            }

            printf("\nHow would you rate your overall experience with our service?\n");
            scanf(" %[^\n]s", costumerFeedback[numberOfSales]);

            correctInput = FALSE;
            // Validate feedback input (only alphabetic characters allowed)
            do {
                for (int i = 0; costumerFeedback[numberOfSales][i] != '\0'; i++) {
                    if (('a' <= costumerFeedback[numberOfSales][i] && costumerFeedback[numberOfSales][i] <= 'z') ||
                        ('A' <= costumerFeedback[numberOfSales][i] && costumerFeedback[numberOfSales][i] <= 'Z')) {
                        correctInput = TRUE;
                    } else {
                        correctInput = FALSE;
                        break;
                    }
                }

                if (!correctInput) {
                    printf("Incorrect Input. Please type with letters: ");
                    scanf(" %[^\n]s", costumerFeedback[numberOfSales]);
                }

            } while (correctInput != TRUE);

        } else if (customerRateInput == 'n') {
            correctInput = TRUE;
            printf("OK, have a nice day\n");
        } else {
            printf("Incorrect input. Please type correct input ('y' or 'n'): ");
            correctInput = FALSE;
        }
    } while (correctInput != TRUE);
}

/*----------------------------------------------------------------------------------------------------------------------*/
/* SORTING YEAR OF MANUFACTURE IN DESCENDING ORDER */
// Function to swap information between cars in the arrays. It will be used the carSortFunction
void swapCars(int i, int j) {
    // Swap carTypes
    char tempType[10];
    strcpy(tempType, carTypes[i]);
    strcpy(carTypes[i], carTypes[j]);
    strcpy(carTypes[j], tempType);

    // Swap yearOfManufacture
    unsigned short tempYear = yearOfManufacture[i];
    yearOfManufacture[i] = yearOfManufacture[j];
    yearOfManufacture[j] = tempYear;

    // Swap remainingCarModel
    int tempRemaining = remainingCarModel[i];
    remainingCarModel[i] = remainingCarModel[j];
    remainingCarModel[j] = tempRemaining;

    // Swap carPrices
    float tempPrice = carPrices[i];
    carPrices[i] = carPrices[j];
    carPrices[j] = tempPrice;
}

// Function to sort cars based on year of manufacture in descending order using the bubble sort algorithm
void caRSort() {
    // Get the number of elements in the yearOfManufacture array
    int n = sizeof(yearOfManufacture) / sizeof(yearOfManufacture[0]);

    // Iterate through the array using the bubble sort algorithm
    for (int i = 0; i < n - 1; i++) {
        // The outer loop performs a pass through the array
        for (int j = 0; j < n - i - 1; j++) {

            if (yearOfManufacture[j] < yearOfManufacture[j + 1]) {
                // If the current year is less than the next year, swap the cars
                swapCars(j, j + 1);
            }
        }
    }
}

void menu_viewCarsStock() {
    caRSort(); // Sort the cars by year of manufacture in descending order
    // Show the user the types of cars after sorting. It will be used in the buy cars function
    printf("\nCars Stocks (sorted by year of manufacture in descending order):\n");
    for (int i = 0; i < sizeof(carTypes) / sizeof(carTypes[0]); i++) {
        printf("%s - Year of manufacture: %d - Car Price: %f - Remaining stock: %d\n",
               carTypes[i], yearOfManufacture[i], carPrices[i], remainingCarModel[i]);
    }
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function of the process of buying cars
void menu_buyCars() {
    printf("\nBuy Cars:\n");
    printf("There are %hd cars available.\n", carAvailable);

    // Check if there are enough cars available
    if (carAvailable == 0) {
        printf("Sorry, there are no more cars available.\n");
        // Terminate the function here
        return;
    }
    //Variables that will need in this function
    unsigned short carNeeded, carType = 0, correctInput = FALSE, giveDiscount;
    float totalPrice;
    char memberOfCarSales;

    // Get customer name with input validation
    printf("\nWhat is your name?\n ");
    do {
        correctInput = TRUE;
        scanf("\n%[^\n]s", customerNames[numberOfSales]);

        for (int i = 0; customerNames[numberOfSales][i] != '\0'; i++) {
            if (!(('a' <= customerNames[numberOfSales][i] && customerNames[numberOfSales][i] <= 'z') ||
                  ('A' <= customerNames[numberOfSales][i] && customerNames[numberOfSales][i] <= 'Z'))) {
                correctInput = FALSE;
                printf("Incorrect Input. Please type the correct input: ");
                break;
            }
        }
    } while (!correctInput);

    // Get customer age with input validation
    printf("\nHow old are you? Age: ");
    while (TRUE) {
        if (scanf("%hd", &costumerAge[numberOfSales]) != 1) {
            printf("Incorrect Input. Please type the correct input: ");
            while (getchar() != '\n');
            continue;
        }
        if (costumerAge[numberOfSales] >= 18) {
            break;
        } else {
            printf("Age must be 18 or older. Please type the correct input: ");
            while (getchar() != '\n');// For clean the buffer
        }
    }


    // Choose car type with input validation
    menu_showCarTypes();
    printf("What type of car do you want to buy?\n");
    while (TRUE) {
        if (scanf("%hd", &carType) != 1 || (carType < 0 || carType > 9)) {
            printf("Incorrect Input. Please type the correct input: ");
            while (getchar() != '\n');// For clean the buffer
            continue;
        }
        if (remainingCarModel[carType] == 0) {
            // Invalid input or no more cars of this model
            printf("Sorry, there are no more cars of this model. Please choose another model.\n");
            scanf("%hd", &carType);
        }
        break;
    }
    // The array tracking the type of car chosen by the customer
    typeOfCarPerSale[numberOfSales] = carType;

    // Choose the number of cars needed with input validation
    printf("\nHow many cars do you need?\n");
    while (TRUE) {
        if (scanf("%hd", &carNeeded) != 1 || carNeeded > remainingCarModel[carType]) {
            printf("Incorrect Input or maybe the amount you want is not available.\nPlease choose again: ");
            while (getchar() != '\n');
            continue;
        }
        break;
    }

    //This array it will track the amount of cars chosen
    carAmountPerSale[numberOfSales] = carNeeded;

    // This array will the total sale of the purchase
    totalPrice = carNeeded * carPrices[carType];

    // This array will update the amount of  the total car available after any purchase
    carAvailable -= carNeeded;

    // This array will update the amount of car available of each type after any purchase
    remainingCarModel[carType] -= carNeeded;

    // Check if the customer is a member of car sales and apply discount if needed
    memberOfCarSales = getCharFromConsole("\nAre you a member of car sales? ('y' or 'n'):\n ", "yn");
    giveDiscount = menu_checkIfDiscountIsNeeded(memberOfCarSales);
    totalPrice = menu_applyDiscount(totalPrice);

    // Print discount outcome and update discountGivenPerSale array
    menu_printDiscountOutcome(giveDiscount);
    discountGivenPerSale[numberOfSales] = giveDiscount;

    // Inform the user about the purchase
    printf("You have bought %d cars.\n", carNeeded);
    printf("Total cost of the cars is %f GBP\n", totalPrice);
    printf("There are %hd cars remaining.\n\n", carAvailable);

    // Request customer feedback
    menu_feedbackRequest();

    // update total sales per car model
    totalSalesPerCarType[carType] += totalPrice;

    // Updated cars have been sold by car model
    carSoldPerType[carType] += carNeeded;

    // Increment the numberOfSales counter
    numberOfSales++;
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function of the sales data
void menu_viewSales() {
    printf("All Sales Data:\n\n");

    //variables to track total sales and cars sold
    float totalSalesValue = 0;
    unsigned int carsSold = 0;

    // Get current time for the date of purchase
    time_t t = time(NULL);
    struct tm tm = *localtime(&t);


    for (int i = 0; i < numberOfSales; i++) {
        int typeofCar = typeOfCarPerSale[i];
        float price = carAmountPerSale[i] * carPrices[typeofCar];

        char discountGivenText[4];
        // text for discount given status
        if (discountGivenPerSale[i] == TRUE) {
            strcpy(discountGivenText, "Yes");
            // Apply discount to the price if applicable
            price *= (1 - DISCOUNT_MEMBER_PERCENTAGE);
        } else {
            strcpy(discountGivenText, "No");
        }


        // Display detailed information about the sale
        printf("\nSale Index: %d | Sale Amount: %f | Type of Car: %s | Car Price: %f |\n"
               "| Number of Cars: %hd | Discount Given: %s | Customer Name: %s | Costumer Age: %hd | "
               "Date of Purchase: %d-%02d-%02d \n",
               i, price, carTypes[typeofCar], carPrices[typeofCar],carAmountPerSale[i],
               discountGivenText, customerNames[i], costumerAge[i], tm.tm_year + 1900, tm.tm_mon +1, tm.tm_mday);

        // Update total sales value and cars sold
        totalSalesValue += price;
        carsSold += carAmountPerSale[i];

        //summary message for each sale
        printf("\n%s has bought %hd units of | %s | for a total of %f GBP ----------------------\n\n",
               customerNames[i] , carAmountPerSale[i], carTypes[typeofCar] , price);

    }
    // Display overall summary of all sales

    printf("\n\n%hd cars have been sold with a total value of %f GBP. There are %hd cars unsold.\n",
           carsSold, totalSalesValue, carAvailable);

}

/*----------------------------------------------------------------------------------------------------------------------*/
void customerFeedback() {
    // Iterate through each sale and display customer feedback
    for (int i = 0; i < numberOfSales; i++) {
        int typeofCar = typeOfCarPerSale[i];
        // Print feedback details for each sale
        printf("%d | Customer Name: %s | Car Type: %s | Costumer Rate: %hd/5 | Feedback: %s \n",
               i,  customerNames[i], carTypes[typeofCar], costumerRate[i], costumerFeedback[i]);
    }
}

/*----------------------------------------------------------------------------------------------------------------------*/
// Function of program exit
void menu_exit() {
    printf("Thank you for using this Cars Sales program. Bye-bye!\n\n");
    // Save sales data to file before exiting
    saveDataToFile();
}

/*----------------------------------------------------------------------------------------------------------------------*/
/* MAIN FUNCTION */
/*----------------------------------------------------------------------------------------------------------------------*/
int main() {
    // Initialize variables and retrieve data from file
    getDataFromFile();
    char userChoice;
    char *inputFileName = "VIEW_CARS.txt";
    char *outputFileName = "VIEW_CARS_encrypted.txt";
    char *key = "Bobby";

    // Perform simple XOR-based file encryption
    XOR(inputFileName, outputFileName, key);

    // Main program loop
    do {
        salesDataPerCarModel();
        menu_greetCustomer();
        menu_showMenu();

        // Get user choice from the menu with input validation
        userChoice = getCharFromConsole("Please choose one: ", "abcdx");

        // Process user choice
        switch (userChoice) {
            case MENU_VIEW_CARS_STOCK:
                menu_viewCarsStock();
                break;

            case MENU_OPTION_BUY_CAR:
                menu_buyCars();
                break;

            case MENU_OPTION_VIEW_SALES:
                menu_viewSales();
                break;

            case MENU_OPTION_COSTUMER_FEEDBACK:
                customerFeedback();
                break;

            case MENU_OPTION_EXIT:
                menu_exit();
                break;
        }

        // Pause the program and wait for user input before clearing the screen
        pauseProgram(userChoice);

    } while (userChoice != MENU_OPTION_EXIT);

    // Display a farewell message
    printf("\n\nHave a good day!\n\n");

    return 0;
}