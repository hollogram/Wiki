```php
/**
 * Handle a registration request for the application.
 *
 * @param Request $request
 * @return \Illuminate\Http\Response
 * @throws Exception
 */
public function register(Request $request)
{
    $this->initRegisterOptions();

    $validator = $this->registerOptions->validator;
    $validator = Validator::make(
        $request->all(), 
        $validator->rules(), 
        $validator->messages(), 
        $validator->attributes());

    if ($validator->fails()) {
        flash()->error($validator->errors()->first());
        return back()->withErrors($validator->errors());
    }

    return DB::transaction(function () use ($request) {
        event(new Registered(
            $user = $this->create($request->all())
        ));

        if ($this->shouldVerifyEmail($user)) {
            $user->load('person');
            $user->generateVerificationToken();
            $user->sendVerificationEmail();
        } else {
            $this->guard()->login($user);
        }

        return $this->registered($request, $user) ?:
            redirect($this->registerOptions->registerRedirect);
    });
}
```