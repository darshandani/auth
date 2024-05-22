<?php

namespace App\Http\Controllers\backend;

use App\Http\Controllers\Controller;
use App\Mail\Adminforgetpassword;
use App\Models\AdminModel;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Str;

class AdminController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        return view('backend.auth_login');
    }

    public function ForgetPasswordview()
    {
        return view('backend.auth_forget');
    }






    /**
     * Show the form for creating a new resource.
     */


    public function doLogin(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required'
        ]);
        $credential = [
            'email' => $request->email,
            'password' => $request->password
        ];
        if (auth()->guard('admin')->attempt($credential)) {
            return redirect()->route('dashboards')->with('success', 'Login Successfully!');
        }
        // dd($credential);
        return redirect()->back()->with('error', 'incorrect email or password. please try again.');
    }


    /**
     * Store a newly created resource in storage.
     */
    public function logout(Request $request)
    {
        auth()->guard('admin')->logout();
        session()->flush();
        session()->regenerate();
        return redirect()->route('authlogin')->with('success', 'You are successfully Logged out.');
    }

    /**
     * Display the specified resource.
     */
    public function ForgetPassword(Request $request)
    {
        $request->validate([
            'email' => 'required|email|exists:admins',
        ]);
        $token = Str::random(64);
        $admin = DB::table('password_reset_tokens')->where('email', $request->email)->get();

        if ($admin) {
            DB::table('password_reset_tokens')->where('email', $request->email)->delete();
        }
        DB::table('password_reset_tokens')->insert([
            'email' => $request->email,
            'token' => $token,
            'created_at' => Carbon::now()
        ]);
        Mail::to($request->email)->send(new Adminforgetpassword($token));
        return redirect()->route('authlogin')->with('success', 'Password reset email sent');
    }


    public function showResetPasswordForm($token)
    {
        // dd($token);
        $object = DB::table('password_reset_tokens')->where('token', $token)->first();
        if (!isset($token)) {
            return redirect()->route('authlogin')->with('error', "Token not exists.");
        }
        return view('backend.auth_reset_password', ['token' => $token, 'email' => $object->email]);
    }
    /**
     * Show the form for editing the specified resource.
     */
    public function ResetPassword(Request $request)
    {
        $request->validate([
            'password' => 'required',
            'password_confirmation' => 'required'
        ]);
        $updatepass = DB::table('password_reset_tokens')->where(['token' => $request->token])->first();
        // dd($updatepass);
        if (!$updatepass) {
            return back()->with('error', 'Invalid Token!');
        }
        $admin = AdminModel::where('email', $updatepass->email)->update(['password' => Hash::make($request->password)]);
        // dd($admin);
        DB::table('password_reset_tokens')->where(['token' => $request->token])->delete();
        return redirect()->route('authlogin')->with('success', 'password changed successfully!');
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, string $id)
    {
        //
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy(string $id)
    {
        //
    }
}
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class Adminforgetpassword extends Mailable
{
    use Queueable, SerializesModels;

    public $token;
    /**
     * Create a new message instance.
     */
    public function __construct($token)
    {
        $this->token = $token;
    }

    /**
     * Get the message envelope.
     */


    public function build()
    {
        return $this->subject('Reset Password Notification')
            ->view('backend.emails.admin_forget_password')
            ->with([
                'token' => $this->token,
            ]);
    }


    
    /**
     * Get the attachments for the message.
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
  
}
<!DOCTYPE html>
<html>

<head>
    <title>Reset Password</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            color: #333;
        }

        .email-container {
            background-color: #ffffff;
            padding: 20px;
            margin: 0 auto;
            max-width: 600px;
            border: 1px solid #dddddd;
            border-radius: 4px;
        }

        .email-header {
            text-align: center;
        }

        .email-header img {
            max-width: 150px;
        }

        .email-body {
            margin-top: 20px;
        }

        .email-footer {
            margin-top: 30px;
            text-align: center;
            font-size: 12px;
            color: #999999;
        }

        .reset-button {
            display: inline-block;
            margin-top: 20px;
            padding: 10px 20px;
            color: #ffffff;
            background-color: #ffb300;
            text-decoration: none;
            border-radius: 4px;
        }
    </style>
</head>

<body>
    <div class="email-container">
        <div class="email-header">
            <img src="https://t3.ftcdn.net/jpg/04/68/17/58/360_F_468175854_cEU6WqxN6Q504R3L7CkDI38Jkh7tYVH4.jpg"
                alt="Your Business Logo">
        </div>
        <div class="email-body">
            <h1>Reset Password Notification</h1>
            {{-- @dd($token) --}}

            <p>You are receiving this email because we received a password reset request for your account.</p>
            <p>If you did not request a password reset, no further action is required.</p>
            <p>Click the button below to reset your password:</p>
            {{-- <a href="{{ url('/reset-password?token=' . $token) }}" class="reset-button">Reset Password</a> --}}
            <a href="{!!route('showResetPasswordForm',['token'=>$token])!!}" class="reset-button">Reset Password</a>
            <p>This password reset link will expire in 60 minutes.</p>
            <p>If you have any questions, please contact our support team at <a
                    href="mailto:support@yourbusiness.com">support@yourbusiness.com</a>.</p>
        </div>
        <div class="email-footer">
            <p>&copy; {{ date('Y') }} Your Taxica. All rights reserved.</p>
        </div>
    </div>
</body>

</html>
