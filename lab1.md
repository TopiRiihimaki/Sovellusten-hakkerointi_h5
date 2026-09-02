## Koodi
```c
#include "stdio.h"

void print_scrambled(char *message)
{
  register int i = 3;
  do {
    printf("%c", (*message)+i);
  } while (*++message);
  printf("\n");
}

int main()
{
  char * bad_message = NULL;
  char * good_message = "Hello, world.";

  print_scrambled(good_message);
  print_scrambled(bad_message);
}
```

### Ratkaisu

Koska bad_message = NULL; se ei kykene ajamaan tuota, Koska bad_message on NULL, se ei osoita mihinkään kelvolliseen muistipaikkaan. Kun koodi yrittää dereferoida (*message), ohjelma kaatua. Ohjelma toimii, jos bad_message annettaisiin jokin merkkijono.
