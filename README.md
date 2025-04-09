Introduction to pipes and related system calls for pipe management

#include <stdio.h> 
#include <unistd.h> 
#include <string.h> 
int main() { 
int fd[2]; // File descriptors for the pipe 
pid_t pid; 
char write_msg[] = "Hello from parent!"; 
char read_msg[100]; 
// Step 2: Create the pipe 
    if (pipe(fd) == -1) { 
        perror("Pipe failed"); 
        return 1; 
    } 
 
    // Step 3: Fork a child process 
    pid = fork(); 
 
    if (pid < 0) { 
        perror("Fork failed"); 
        return 1; 
    } 
 
    if (pid > 0) { // Parent process 
        // Step 4: Close unused read end 
        close(fd[0]); 
 
        // Step 5: Write to the pipe 
        write(fd[1], write_msg, strlen(write_msg) + 1); 
        close(fd[1]); // Close write end after writing 
 
    } else { // Child process 
        // Step 4: Close unused write end 
        close(fd[1]); 
 
        // Step 5: Read from the pipe 
        read(fd[0], read_msg, sizeof(read_msg)); 
        printf("Child received: %s\n", read_msg); 
        close(fd[0]); // Close read end after reading 
    } 
 
    return 0; 
} 

Framing Protocol: WAP for Character Count 

#include <stdio.h> 
#include <string.h> 
// Function to simulate the sender 
void sender(const char *messages[], int num_messages, char frames[][100]) { 
for (int i = 0; i < num_messages; i++) { 
int length = strlen(messages[i]);    // Calculate the message length 
frames[i][0] = length;
               //
 First byte is the length 
strcpy(frames[i] + 1, messages[i]); // Copy the message after the length 
} 
} 
// Function to simulate the receiver 
void receiver(char frames[][100], int num_frames) { 
for (int i = 0; i < num_frames; i++) { 
int length = frames[i][0]; // Read the first byte as length 
printf("Frame %d (Length: %d): %.*s\n", i + 1, length, length, frames[i] + 1); 
} 
} 
int main() { 
const char *messages[] = {"Hello", "World", "Character Count Protocol"}; 
int num_messages = sizeof(messages) / sizeof(messages[0]); 
char frames[10][100]; // Array to store frames 
// Step 1: Sender creates frames 
sender(messages, num_messages, frames); 
// Step 2: Receiver processes frames 
printf("Receiver Output:\n"); 
receiver(frames, num_messages); 
return 0; 
}

WAP to Implement Framing Protocol: Byte Stuffing

#include <stdio.h> 
#include <string.h> 
 
#define SOF '@'   // Start-of-frame marker 
#define ESC '#'   // Escape character 
 
// Function to perform byte stuffing at the sender's side 
void sender(const char *message, char *stuffed_frame) { 
    int j = 0; 
    stuffed_frame[j++] = SOF; // Add SOF at the start of the frame 
 
    for (int i = 0; message[i] != '\0'; i++) { 
        if (message[i] == SOF || message[i] == ESC) { 
            stuffed_frame[j++] = ESC; // Add escape character 
        } 
        stuffed_frame[j++] = message[i]; // Add the actual character 
    } 
 
    stuffed_frame[j++] = SOF; // Add SOF at the end of the frame 
    stuffed_frame[j] = '\0';  // Null-terminate the stuffed frame 
} 
 
// Function to perform byte unstuffing at the receiver's side 
void receiver(const char *stuffed_frame, char *original_message) { 
    int j = 0; 
    for (int i = 1; stuffed_frame[i] != SOF; i++) { // Skip the initial SOF 
        if (stuffed_frame[i] == ESC) { 
            i++; // Skip the escape character 
        } 
        original_message[j++] = stuffed_frame[i]; 
    } 
    original_message[j] = '\0'; // Null-terminate the original message 
} 
 
int main() { 
    const char *message = "Hello @World# Protocol"; 
    char stuffed_frame[100], original_message[100]; 
 
    // Step 1: Perform byte stuffing 
    sender(message, stuffed_frame); 
    printf("Stuffed Frame: %s\n", stuffed_frame); 
 
    // Step 2: Perform byte unstuffing 
    receiver(stuffed_frame, original_message); 
    printf("Original Message: %s\n", original_message); 
 
    return 0; 
}

WAP to Implement Framing Protocol: Bit Stuffing

#include <stdio.h> 
#include <string.h> 
 
#define FLAG "01111110" 
 
// Function to perform bit stuffing at the sender's side 
void sender(const char *data, char *stuffed_data) { 
    int count = 0, j = 0; 
 
    // Add the flag sequence at the start of the frame 
    strcpy(stuffed_data, FLAG); 
    j += strlen(FLAG); 
 
    for (int i = 0; data[i] != '\0'; i++) { 
        if (data[i] == '1') { 
            count++; 
        } else { 
            count = 0; 
        } 
 
        // Add the current bit to the stuffed data 
        stuffed_data[j++] = data[i]; 
 
        // Stuff a '0' after five consecutive '1's 
        if (count == 5) { 
            stuffed_data[j++] = '0'; 
            count = 0; 
        } 
    } 
 
    // Add the flag sequence at the end of the frame 
    strcpy(stuffed_data + j, FLAG); 
    j += strlen(FLAG); 
    stuffed_data[j] = '\0'; // Null-terminate the stuffed data 
} 
 
// Function to perform bit unstuffing at the receiver's side 
void receiver(const char *stuffed_data, char *original_data) { 
    int count = 0, j = 0; 
 
    // Skip the initial flag sequence 
    int start = strlen(FLAG); 
 
    for (int i = start; stuffed_data[i] != '\0'; i++) { 
        // Stop at the final flag sequence 
        if (strncmp(stuffed_data + i, FLAG, strlen(FLAG)) == 0) { 
            break; 
        } 
 
        if (stuffed_data[i] == '1') { 
            count++; 
        } else { 
            count = 0; 
        } 
 
        // Add the current bit to the original data 
        original_data[j++] = stuffed_data[i]; 
 
        // Skip the stuffed '0' after five consecutive '1's 
        if (count == 5 && stuffed_data[i + 1] == '0') { 
            i++; 
            count = 0; 
        } 
    } 
    original_data[j] = '\0'; // Null-terminate the original data 
} 
 
int main() { 
    const char *data = "0111111011111100001111111"; // Input binary string 
    char stuffed_data[100], original_data[100]; 
 
    // Step 1: Perform bit stuffing 
    sender(data, stuffed_data); 
    printf("Stuffed Data: %s\n", stuffed_data); 
 
    // Step 2: Perform bit unstuffing 
    receiver(stuffed_data, original_data); 
    printf("Original Data: %s\n", original_data); 
 
    return 0; 
}

WAP to Implement Error Detection: LRC and Checksum

Code Implementation for LRC: 

#include <stdio.h> 
#include <string.h> 
 
void calculateLRC(char data[][9], int rows, char *lrc) { 
    int colSum[8] = {0}; 
 
    // Calculate LRC 
    for (int col = 0; col < 8; col++) { 
        for (int row = 0; row < rows; row++) { 
            colSum[col] ^= (data[row][col] - '0'); // XOR each bit column-wise 
        } 
        lrc[col] = colSum[col] + '0'; // Convert back to character 
    } 
    lrc[8] = '\0'; // Null-terminate the LRC 
} 
 
int main() { 
    char data[4][9] = { // Example binary data (8 bits per block) 
        "11001101", 
        "10101010", 
        "11110000", 
        "00001111" 
    }; 
 
    char lrc[9]; 
    calculateLRC(data, 4, lrc); 
 
    printf("Input Data Blocks:\n"); 
    for (int i = 0; i < 4; i++) { 
        printf("%s\n", data[i]); 
    } 
 
    printf("LRC: %s\n", lrc); 
 
    return 0; 
} 

Code Implementation for Checksum: 

#include <stdio.h> 
#include <string.h> 
 
// Function to calculate checksum 
unsigned int calculateChecksum(int data[], int n) { 
    unsigned int sum = 0; 
 
    // Add all data blocks 
    for (int i = 0; i < n; i++) { 
        sum += data[i]; 
    } 
 
    // Calculate the 1's complement of the sum 
    unsigned int checksum = ~sum; 
    return checksum; 
} 
 
int main() { 
    int data[] = {0x1234, 0x5678, 0x9ABC, 0xDEF0}; // Example data (16-bit blocks) 
    int n = sizeof(data) / sizeof(data[0]); 
 
    // Calculate checksum 
    unsigned int checksum = calculateChecksum(data, n); 
 
    printf("Input Data Blocks:\n"); 
    for (int i = 0; i < n; i++) { 
        printf("0x%X\n", data[i]); 
    } 
 
    printf("Checksum: 0x%X\n", checksum); 
 
    return 0; 
}
