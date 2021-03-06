source: http://laraveldaily.com/laravel-login-email-username-one-field/ 

#Laravel Auth: Аутентификация через логин или email (используя одно поле).

Лайфхак для вас, ребята. Что если в вашем проекте пользователи должны уметь логиниться не только по email, но также и через другие поля, к примеру используя поле "username" или "user_id" или любое другое? По умолчанию Laravel позволяет использовать только одно поле, например "email". На самом деле это легко изменить.

Давайте представим что у вас уже есть поле username в таблице пользователей и вам нужно проверять результат формы логина. Вот только в форме всего 2 инпута - логин и пароль, а логином может быть как email так и username. Так как же проверить их вместе?

Для этого вам понадобится добавить несколько строчек кода в файл контроллера ```app/Http/Auth/LoginController.php``` и переопределить метод ```login()```:


```
public function login(Request $request)
{
    $this->validate($request, [
        'login'    => 'required',
        'password' => 'required',
    ]);
 
    $login_type = filter_var($request->input('login'), FILTER_VALIDATE_EMAIL ) 
        ? 'email' 
        : 'username';
 
    $request->merge([
        $login_type => $request->input('login')
    ]);
 
    if (Auth::attempt($request->only($login_type, 'password'))) {
        return redirect()->intended($this->redirectPath());
    }
 
    return redirect()->back()
        ->withInput()
        ->withErrors([
            'login' => 'These credentials do not match our records.',
        ]);
    } 
 
}
```

Давайте рассмотрим подробнее:

```
$login_type = filter_var($request->input('login'), FILTER_VALIDATE_EMAIL ) 
  ? 'email' 
  : 'username';

```

Эта строчка проверяет что ввел пользователь: email или что то другое - то, что будет интерпретировано как логин.

```
$request->merge([
    $login_type => $request->input(‘login')
]);

```


Код выше добавляет переменную в массив $request - по умолчанию у нас есть переменная 'login', но нам надо либо `email` либо `username` - так вот этот код добавляет недостающий этот функционал.


```
if (Auth::attempt($request->only($login_type, 'password'))) {
    return redirect()->intended($this->redirectPath());
}

```

В итоге мы проверяем учетные данные, но только что то одно, то поле которое нам нужно. Нужные данные приходят из переменной $login_type.

Все очень просто, не правда ли?



