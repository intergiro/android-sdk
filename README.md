# Intergiro Android SDK

## Compatibility

Android mobile SDK supports:

- minSdkVersion 23 and above.
- compileSdkVersion and targetSdkVersion 31 and above. Please refer to this [requirement](https://developer.android.com/google/play/requirements/target-sdk).

## Github package token generation

Due to the nature of github packages to fetch our SDK you need to generate your own github token. One of the ways would be to generate classical github token with "read:packages"
scope. More
info [here](https://docs.github.com/en/enterprise-server@3.6/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token).

## Getting your github account id

Due to the nature of github packages to fetch our SDK you need to get ID of your personal account. One of the ways would be to make request
to https://api.github.com/users/your_github_user_name replacing your_github_user_name with your actual github user name. The
request's respond contains "id" field which refers to
ID of account with your_github_user_name name.

## Installation

1. Download [Google Play services SDK supplement dependency](https://developers.google.com/pay/issuers/apis/push-provisioning/android/releases) and unzip the SDK supplement to
   your project's folder. You will have a directory structure similar to `./tapandpay_sdk/com/google/android/gms/...`. Make sure the path to the SDK supplement aligns with the
   path you configure in your Gradle configuration later.
2. Add it to your `settings.gradle` file:

```
dependencyResolutionManagement {
    // ...
    repositories {
        // ...
        maven { url "https://jitpack.io" }
        maven { url "file:${project.rootDir}/tapandpay_sdk/" } // should be replaced with path to your Google Play services SDK supplement.
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/intergiro/android-sdk")
            credentials {
                username = id_of_your_github_account
                password = "your_github_packages_token"
            }
        }
    }
}
```

or if you use an older version of gradle add it to your root `build.gradle` file:

```
allprojects {
    // ...
    repositories {
        // ...
        maven { url "https://jitpack.io" }
        maven { url "file:${project.rootDir}/tapandpay_sdk/" } // should be replaced with path to your Google Play services SDK supplement.
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/intergiro/android-sdk")
            credentials {
                username = id_of_your_github_account
                password = "your_github_packages_token"
            }
        }
    }
}
```

3. Add dependency to your app `build.gradle` specifying the latest version

```
dependencies {
    implementation 'com.intergiro.android:sdk:x.y.z'
}
```

## Configuration

1. Add public key to `AndroidManifest.xml`. To get your public key please contact <mobile-sdk@intergiro.com>. Note: *SDK will throw RuntimeException in case if public key was not
   set.*

```
<manifest>
    <application>
        <meta-data
            android:name="com.intergiro.android.sdk.PUBLIC_KEY"
            android:value="{your_public_key_value}" />
    </application>
</manifest>
```

2. SDK will be automatically initialized by [Jetpack App Startup](https://developer.android.com/topic/libraries/app-startup).
3. (*Optional*) You can set your custom theme to `IntergiroWebActivity` (`NoActionBar` variants are preferred).

```
<activity
    android:name="com.intergiro.android.sdk.ui.IntergiroWebActivity"
    tools:replace="android:theme"
    android:theme="@style/{your_custom_theme}" />
```

#### Permissions

We include the [INTERNET](https://developer.android.com/reference/android/Manifest.permission#INTERNET) permission by default as we need it to make network requests:

```
<uses-permission android:name="android.permission.INTERNET"/>
```

#### ProGuard / R8

SDK should work without any custom proguard configuration.

## Usage

To start Intergiro consent flow you should use `IntergiroUserSession` ([how this works](https://developer.android.com/training/basics/intents/result)). This class will
start `IntergiroWebActivity` where interaction with Intergiro 3D API happens.

Flow will end up with `IntergiroUserSessionResult`. Result has optional `String` payload (usually it's json) and it could be one of 3 types:

* `IntergiroUserSessionResult.Completed` - Flow completed normally
* `IntergiroUserSessionResult.Canceled` - User ended flow by pressing/swiping back.
* `IntergiroUserSessionResult.Error` - Error happened, like connection error.

`Error` payload has next format:

```
{
    code: int,
    domain: string, (unexpected_error|connection_error)
    description: string
}
```

## Example of usage

```
class SampleActivity : AppCompatActivity(), ActivityResultCallback<IntergiroUserSessionResult> {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //... app code ...

        val launcher = registerForActivityResult(IntergiroUserSession(), ::onActivityResult)
        someBtn.setOnClickListener { launcher.launch(token) }
    }

    override fun onActivityResult(result: IntergiroUserSessionResult) {
        when (result) {
            is IntergiroUserSessionResult.Completed -> {
                //Handle result
            }

            is IntergiroUserSessionResult.Canceled -> {
                //Handle result
            }

            is IntergiroUserSessionResult.Error -> {
                //Handle result
            }
        }
    }
}
```
