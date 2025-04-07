#include <iostream>     // For input/output operations
#include <fstream>      // For file operations
#include <cctype>       // For character classification functions like isalpha, isdigit
#include <cstring>      // For string manipulation functions like strcpy

using namespace std;

// Character classes
#define LETTER 0
#define DIGIT 1
#define UNKNOWN 99
#define END_OF_FILE -1

// Token codes
#define INT_LIT 10
#define IDENT 11
#define ASSIGN_OP 20
#define ADD_OP 21
#define SUB_OP 22
#define MULT_OP 23
#define DIV_OP 24
#define LEFT_PAREN 25
#define RIGHT_PAREN 26

int charClass;            // To hold character classification
char lexeme[100];         // To store the current lexeme (string of characters)
char nextChar;            // To hold the next character from input
int lexLen = 0;           // Length of the lexeme
int nextToken;            // Token for the current lexeme
ifstream in_fp;           // Input file stream

// Function declarations
void addChar();
void getChar();
void getNonBlank();
int lookup(char ch);
int lex();

// Main driver function
int main() {
    // Try to open the file "front.in"
    in_fp.open("front.in");
    if (!in_fp.is_open()) {
        cout << "ERROR - cannot open front.in" << endl;
    } else {
        getChar(); // Read the first character
        do {
            lex();  // Perform lexical analysis
        } while (nextToken != END_OF_FILE); // Until EOF
    }
    return 0;
}

// Adds the current character to the lexeme array
void addChar() {
    if (lexLen <= 98) {
        lexeme[lexLen++] = nextChar;
        lexeme[lexLen] = '\0'; // Null-terminate the string
    } else {
        cout << "Error - lexeme is too long" << endl;
    }
}

// Reads the next character from the input file and sets its class
void getChar() {
    nextChar = in_fp.get();
    if (in_fp.eof()) {
        charClass = END_OF_FILE;
    } else if (isalpha(nextChar)) {
        charClass = LETTER;
    } else if (isdigit(nextChar)) {
        charClass = DIGIT;
    } else {
        charClass = UNKNOWN;
    }
}

// Skips over any whitespace characters
void getNonBlank() {
    while (isspace(nextChar))
        getChar();
}

// Determines if a character is an operator or special symbol
int lookup(char ch) {
    switch (ch) {
        case '(': addChar(); nextToken = LEFT_PAREN; break;
        case ')': addChar(); nextToken = RIGHT_PAREN; break;
        case '+': addChar(); nextToken = ADD_OP; break;
        case '-': addChar(); nextToken = SUB_OP; break;
        case '*': addChar(); nextToken = MULT_OP; break;
        case '/': addChar(); nextToken = DIV_OP; break;
        default:  addChar(); nextToken = END_OF_FILE; break;
    }
    return nextToken;
}

// Lexical analyzer function
int lex() {
    lexLen = 0;
    getNonBlank(); // Skip whitespaces

    switch (charClass) {
        // Identifier: starts with letter, may contain letters/digits
        case LETTER:
            addChar();
            getChar();
            while (charClass == LETTER || charClass == DIGIT) {
                addChar();
                getChar();
            }
            nextToken = IDENT;
            break;

        // Integer literal: sequence of digits
        case DIGIT:
            addChar();
            getChar();
            while (charClass == DIGIT) {
                addChar();
                getChar();
            }
            nextToken = INT_LIT;
            break;

        // Operator or special symbol
        case UNKNOWN:
            lookup(nextChar);
            getChar();
            break;

        // End of file
        case END_OF_FILE:
            nextToken = END_OF_FILE;
            strcpy(lexeme, "EOF");
            break;
    }

    // Output the result
    cout << "Next token is: " << nextToken << ", Next lexeme is " << lexeme << endl;
    return nextToken;
}
