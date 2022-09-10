# Catch cpp exceptions in cgo 在 cgo 中捕获 cpp 异常

## Add the following code to your program 将下面代码添加到你的程序中
``` cpp
//Add a custom exception catch action in the code below
//在下面的代码中添加自定义的异常捕获操作
#define Try(Func)                                          \
    try {                                                  \
        _err_flag = 0;                                     \
        Func;                                              \
    } catch (cv::Exception & ex) {                         \
        _err_flag = 1;                                     \
        _err_len = strlen(ex.what());                      \
        _err_str = (char*)realloc(_err_str, _err_len + 1); \
        _err_str[_err_len] = '\0';                         \
        strcpy(_err_str, ex.what());                       \
    } catch (std::exception & ex) {                        \
        _err_flag = 1;                                     \
        _err_len = strlen(ex.what());                      \
        _err_str = (char*)realloc(_err_str, _err_len + 1); \
        _err_str[_err_len] = '\0';                         \
        strcpy(_err_str, ex.what());                       \
    } catch (...) {                                        \
        char err[] = "unknow err\0";                       \
        _err_flag = 1;                                     \
        _err_len = strlen(err);                            \
        _err_str = (char*)realloc(_err_str, _err_len + 1); \
        _err_str[_err_len] = '\0';                         \
        strcpy(_err_str, err);                             \
    }

//The following is the interaction code of c and go
//下面是c和go的交互代码
int _err_flag; 
char* _err_str;
int _err_len;
void ErrInit()
{
    _err_flag = 0;
    _err_len = 0;
    _err_str = (char*)malloc(1);
}
int Erred()
{
    return _err_flag;
}
const char* Err()
{
    return _err_str;
}
int ErrLen()
{
    return _err_len;
}

```

``` go

//below is the code in go
//下面是go中的代码
/*
#include <stdlib.h>
void ErrInit();
int Erred();
const char* Err();
int ErrLen();
*/
import "C"

func init() {
	C.ErrInit()
}

func cerr() error {
	if C.Erred() == 1 {
		l := C.ErrLen()
		s := C.GoStringN(C.Err(), l)
		return errors.New(s)
	}
	return nil
}

```

## Call as follows 按照下面的方法调用

- Just try({})  in cpp; 在cpp中只需要Try()一下
- Just cerr() in go; 在go中只需要cerr()一下
``` cpp

void SayHello(){
    Try({
        //do something
    })
}
```

``` go

func main(){

    C.SayHello()

    //If calling a method in cgo throws an exception, the next cerr() will return the corresponding error
    //如果 调用 cgo 中的方法抛出异常，那么下一个cerr()就会返回相应错误
    if err := cerr(){
		return log.Println(err)
	}
}

```
