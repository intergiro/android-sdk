# Intergiro Android SDK

## Compatibility

The Android mobile SDK supports:

- minSdkVersion 23 and above.
- compileSdkVersion and targetSdkVersion 31 and above. Please refer to
  this [requirement](https://developer.android.com/google/play/requirements/target-sdk).

## Github package token generation

To fetch our SDK from GitHub packages, you’ll need to generate your own GitHub token. One method is
to generate a standard GitHub token with "read:packages"
scope. For more information, click [here](https://docs.github.com/en/enterprise-server@3.6/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token)
.

## Getting your github account id

To fetch our SDK from GitHub packages, you’ll need to retrieve the ID of your personal account. One
method is to send a request to  https://api.github.com/users/your_github_user_name. Replace
your_github_user_name with your actual GitHub username. The
Response will contain an "id" key that corresponds to your account’s ID.

## Installation

1. Download
   the [Google Play services SDK supplement dependency](https://developers.google.com/pay/issuers/apis/push-provisioning/android/releases)
   and unzip it to your project’s folder. This will create a directory structure
   like `./tapandpay_sdk/com/google/android/gms/...`. Ensure that the SDK supplement’s path aligns
   with what you later configure in your Gradle setup.
2. Add the following to your `settings.gradle` file:

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

1. Add your public key to the `AndroidManifest.xml`. To get your public key please
   contact <mobile-sdk@intergiro.com>. Note: If the public key isn’t set, the SDK will throw a
   ‘RuntimeException’.

```
<manifest>
    <application>
        <meta-data
            android:name="com.intergiro.android.sdk.PUBLIC_KEY"
            android:value="{your_public_key_value}" />
    </application>
</manifest>
```

2. SDK will be automatically initialized
   by [Jetpack App Startup](https://developer.android.com/topic/libraries/app-startup).
3. (*Optional*) You can set your custom theme to `IntergiroWebActivity` (`NoActionBar` variants are
   preferred).

```
<activity
    android:name="com.intergiro.android.sdk.ui.IntergiroWebActivity"
    tools:replace="android:theme"
    android:theme="@style/{your_custom_theme}" />
```

#### Permissions

We include
the [INTERNET](https://developer.android.com/reference/android/Manifest.permission#INTERNET)
permission by default as we need it to make network requests:

```
<uses-permission android:name="android.permission.INTERNET"/>
```

#### ProGuard / R8

SDK should work without any custom proguard configuration.

## Usage

To initiate the Intergiro consent flow,
utilize `IntergiroUserSession` ([how this works](https://developer.android.com/training/basics/intents/result))
. This class will launch `IntergiroWebActivity` where you can interact with the Intergiro 3d API.

The flow concludes with an`IntergiroUserSessionResult`. This result contains an optional `String`
payload (typically it's JSON) and can be one of three types:

* `IntergiroUserSessionResult.Completed` - Flow completed normally
* `IntergiroUserSessionResult.Canceled` - User ended flow by pressing/swiping back.
* `IntergiroUserSessionResult.Error` - Error happened, like connection error.

The `Error` payload has the following structure::

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

## License

Intergiro Android SDK is released under the [MIT license](LICENSE.md).