
## World Tour
 Given a file with hundreds of coordinates, find a secret message with the initials of all
    the countries which those coordinates correspond to.

```python
from geopy.geocoders import Nominatim
geolocator = Nominatim(user_agent='pasok')

f= open("enc.txt", "r")

line = f.read()

coordinate_list = [x.replace('(','').replace(')','') for x in line.split(")(")]

output_file = open("test.txt","w+")

for coordinate in coordinate_list:
    output_file.write(geolocator.reverse(coordinate,language="en-gb").raw["address"]["country"])
    output_file.write('\n')

output_file.seek(0,0)

print("nactf{",end="")
for x in output_file :
    print(x[0],end="")
print("}")

output_file.close()
f.close()
```

## Dr. J's Vegetable Factory 1/2 🥕
Given comma separated vegetables, provide indexes which will be used to sort
    the array. (the vegetable in the provided index will swap with the next
    vegetable on the array)

**C program which will sort the vegetables:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct dynamic_array {

    void * arr ;
    int N;
};

int isSorted(char * arr [], int N) {

    int r = 1;
    for(int i = 0; i < N - 1 && r == 1 ; i++) {
        if (strcmp(arr[i],arr[i+1]) > 0) {
            r = 0;
        }
    }
    return r;
}

struct dynamic_array fun (int N, char ** arr) {

    int j = 0;
    int * indexes = NULL;

    while (!isSorted(arr, N)) {
        for (int i = 0; i < N - 1 ; i++) {
            if (strcmp(arr[i],arr[i+1] ) > 0) {
                indexes = realloc(indexes,sizeof(int) * (j+1));
                *(indexes+j) = i;
                char * tmp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = tmp;
                j++;
            }
        }
    }
    return (struct dynamic_array) {.N = j, .arr = indexes};
}

struct dynamic_array parse_vegetables(char * str){

    char ** mystrings = NULL;
    char * token = strtok(str, ",");
    int j;

    for(j = 0; token != NULL; j++){
        mystrings = realloc(mystrings, sizeof (char*) * (j+1));
        if(j > 0)  {
            mystrings[j] = token+1;
        }
        else {
            mystrings[j] = token;
        }
        token = strtok(NULL,",");
    }
    return (struct dynamic_array) {.N = j , .arr=mystrings};
}

int main (int argc, char *argv[]) {

    struct dynamic_array mystrings, arr_indexes;

    mystrings = parse_vegetables(argv[1]);
    arr_indexes = fun(mystrings.N,mystrings.arr);

    for (int i = 0; i < arr_indexes.N ; i++) {

         printf("%d", *(((int * )arr_indexes.arr)+i));

        if (i < arr_indexes.N - 1 ) {
          putchar(' ');
        }
    }
    putchar('\n');
    free(arr_indexes.arr);
    free(mystrings.arr);

    return 0;
}

```
**Python script used to interact with the challenge server:**

```python
import socket
import subprocess

# Open socket to interact with challenge server:
s = socket.socket()
s.connect(("challenges.ctfd.io",30267))

# Receive initial info
info = s.recv(4096).decode("utf-8")
print(info)
# Choose challenge
s.send(b'2\n')

i = 0;
while i < 4:
    response = s.recv(4096).decode("utf-8")
    print(response)
    # Get veggies 🥬🥕🌽🍆🥦🥒🥑🍄
    if (i == 0):
        veggies = response.split("\n\n")[1]
    else:
        veggies = response.split("🍄")[-1]

    veggies=veggies.strip()
    # Send the robot instructions
    a = subprocess.run(["./a.out",veggies], capture_output=True)
    print(a.stdout.decode("utf-8"))
    s.send(a.stdout)

    i += 1


```
