Utwórz Dockerfile, który będzie kompilował poniższy kod napisany w go:
```go
package main
import "fmt"
func main() {
    fmt.Println("hello world")
}
```

następnie skonfiguruj skompilowany kod jako `ENTRYPOINT` \
wybudowany obraz powinien być tzn. `distroless`

Efekt końcowy ma być taki, że użytkownik po uruchomieniu kontenera dostanie komunikat: `hello world` i kontener zakończy działanie.

---
Teraz zmodyfikuj Dockerfile tak, żeby kontener nie wychodził, ale co 1 sekunde pisał na ekranie: `hello world`
> TIP: w tej części trzeba zrezygnować z `distroless`
