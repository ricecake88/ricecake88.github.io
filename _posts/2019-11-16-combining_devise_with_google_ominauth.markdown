---
layout: post
title:      "Combining Devise with Google OminAuth"
date:       2019-11-16 17:15:05 -0500
permalink:  combining_devise_with_google_ominauth
---


On the recommendation of the section lead, I decided to try using "Devise" to do the authentication for gem users. 

There are a lot of great user guides of how to use the devise gem, especially their own website: http://devise.plataformatec.com.br/

When it came to incorporating how to add an additional sign on from a third party provider for Google, it was a bit trickier and required a lot more digging around. And even when I did find some instructions from several places, I had to stitch them together, and they didn't really explain why certain code was used or the relation to existing code. There are a few sites that did outline how one might add multiple providers (ie. Google, Facebook, etc.) but I wanted to just focus on Google for my project.



Below are the instructions needed to use Devise individual sign-up / login with Google sign in, separated by sections. 


## Installing and Configuring Devise

1. Create a brand new rails project by entering the following command: <br/>
`$rails new <project_name>`<br/>

2.  Install the devise gem by updating your Gemfile with the following line: 
`gem 'devise'`

3. Enter into the following into the command to install the devise gem: 
`$bundle update`

4. Choose where you are going to set up your root page. You can either create a new model/migration to work in conjunction with your resource to set a page as your root. Since this is for a new project, I decided that my root would be` 'pages#index`'. This required I set up `app/controllers/pages_controller.rb` with the following content:
```
class PagesController < ApplicationController
    def index
    end
end
```

5. Add some text into the root view file app/views/pages/index.html.erb so when a user logs in, they can tell they have logged in. Like, "Welcome!"

6. Now we need to install devise in your project using a generate. Enter the following: `$rails generate devise:install`

7. The above command will create the devise model and configuration file, in the following paths along with this printout:  
```
Running via Spring preloader in process 13854
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
```

8. In addition, it will print out the following instructions:
Some setup you must do manually if you haven't yet:
```
  1. Ensure you have defined default url options in your environments files. Here is an example of default_url_options appropriate for a development environment in config/environments/development.rb:
       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
     In production, :host should be set to the actual host of your application.
  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:
        root to: "home#index"
  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:
       <p class="notice"><%= notice %>
       <p class="alert"><%= alert %>
  4. You can copy Devise views (for customization) to your app by running:
       rails g devise:views
```


9. Follow steps 1 through 4 of the instructions output. Note that for step 4, if you want your views to be located and associated with a different folder other than "devise", for example "users", you would want to enter in the following format:`$rails generate devise:views <name_of_folder_other_than_devise>`


10. (Optional Step) If you plan on making modifications to the controllers, then you will want to enter the following command. If not, this step can be skipped (which is what I did for my project). Similarly, the folder in which the controllers are located can be named differently than "devise" and would follow the following format: `$rails generate devise:controllers <name_of_folder_other_than_devise>` If you want the controllers to be under "devise" than no name needs to be specified after "devise::controllers".

11. When using the generator to install devise from step 5, it installs an initializer that describes all of Devises' configuration options. Devise says '**It is imperative that you take a look at it**.'. This file is: `'config/initializers/devise.rb'`

12. Devise then needs a model that matches up with the equivalent of what you would like to name the users of your application to be called, typically called "User". Entering the following command will create this model and configures it with the default Devise modules.`$rails generate devise <MODEL_NAME>`

13. The above command automatically creates and adds a lot of files and code. Here are a few items that it adds:
 * the generator configures the routes file in "`config/routes.rb`" (if your model and migration is :users).
 *  `devise_for :users`
 *  In addition, it creates all the devise views under "views/users"
 *  Creates a migration in which it adds devise to user, adding the fields for email, encrypted password, tokens, etc.

14.  Run the latest migrations by entering: `$rake db:migrate`

15.  Verify the routes that were created by entering the following:`$rake routes`The routes should look like the following:
```
    Prefix Verb   URI Pattern                                                                              Controller#Action
                     root GET    /                                                                                        pages#index
         new_user_session GET    /users/sign_in(.:format)                                                                 devise/sessions#new
             user_session POST   /users/sign_in(.:format)                                                                 devise/sessions#create
     destroy_user_session DELETE /users/sign_out(.:format)                                                                devise/sessions#destroy
        new_user_password GET    /users/password/new(.:format)                                                            devise/passwords#new
       edit_user_password GET    /users/password/edit(.:format)                                                           devise/passwords#edit
            user_password PATCH  /users/password(.:format)                                                                devise/passwords#update
                          PUT    /users/password(.:format)                                                                devise/passwords#update
                          POST   /users/password(.:format)                                                                devise/passwords#create
 cancel_user_registration GET    /users/cancel(.:format)                                                                  devise/registrations#cancel
    new_user_registration GET    /users/sign_up(.:format)                                                                 devise/registrations#new
   edit_user_registration GET    /users/edit(.:format)                                                                    devise/registrations#edit
        user_registration PATCH  /users(.:format)                                                                         devise/registrations#update
                          PUT    /users(.:format)                                                                         devise/registrations#update
                          DELETE /users(.:format)                                                                         devise/registrations#destroy
                          POST   /users(.:format)                                                                         devise/registrations#create
       rails_service_blob GET    /rails/active_storage/blobs/:signed_id/*filename(.:format)                               active_storage/blobs#show
rails_blob_representation GET    /rails/active_storage/representations/:signed_blob_id/:variation_key/*filename(.:format) active_storage/representations#show
       rails_disk_service GET    /rails/active_storage/disk/:encoded_key/*filename(.:format)                              active_storage/disk#show
update_rails_disk_service PUT    /rails/active_storage/disk/:encoded_token(.:format)                                      active_storage/disk#update
     rails_direct_uploads POST   /rails/active_storage/direct_uploads(.:format)                                           active_storage/direct_uploads#create
```


16. To see and understand the devise gem more in depth, visit this [page](https://www.sitepoint.com/devise-authentication-in-depth/).

## Installing OmniAuth and Google to work with Devise (Separate sign-in AND Option to Log In with Google)

### Installing the Required Gems

1. Modify the Gemfile by add omniauth-google-oauth2, thin, and dotenv-rails. *  *Omniauth-google-oauth2* is the gem required to install the Google Omniauth files. 
*  *Thin* allows one to launch your application securely using https. 
*  *Dotenv-rails* is the gem that supports reading .env files (where the credentials will be stored)
```
    gem 'omniauth-google-oauth2'
    gem 'thin'
    gem 'dotenv-rails'
```


2. Enter the following to install the above gems.`$bundle update`

## Add OmniAuth to model Users 
3. Going from the above Devise instructions, since our model for users is named User, we need to add OmniAuth columns to the model to support the fields required for OmniAuth. Make a note of all the properties you might like to store from Google and add it to the end of the following command. Or, you can just leave the command as is:`$bin/rails g migration AddOmniauthToUsers provider:string uid:string token:string expires_at:integer expires:boolean refresh_token:string`

4. Run the migration to add the omniauth columns to the User model`$bin/rails db:migrate`

5. Having added those new fields to the User model, we need to modify the model to support and handle OmniAuth. Currently, the '`app/models/user.rb`' look like the below with the devise modifications.
```
	class User < ApplicationBase
			devise :database_authenticatable, :registerable,
						 :recoverable, :rememberable, :trackable, :validatable
	end
```


 Under 'app/models/user.rb', modify the class to look like the following in order to support and handle OmniAuth.
```
	class User < ApplicationBase
			devise :database_authenticatable, :registerable,
						 :recoverable, :rememberable, :trackable, :validatable
						 :omniauthable, :omniauth_providers => [:google_oauth2]
	       end

           def self.from_omniauth(auth)
                # Create a User record or update it based on Google and the UID from Google.
                where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
                    user.email = auth.info.email
                    user.password = Devise.friendly_token[0,20]
                    user.token = auth.credentials.token
                    user.expires = auth.credentials.expires
                    user.expires_at = auth.credentials.expires_at
                    user.refresh_token = auth.credentials.refresh_token
                end
            end
   end
```


6. Adding "devise :omniauthable", is to tell Devise that it's an Omniauthable model and has other functionality on top of the normal devise methods. Also added the class method "`from_omniauth`". We receive a hash that includes the authentication parameter which we pass to the method ("auth" is the variable in the above example) that we receive from Google. By using that hash we will then be able to create a User from it.

### Getting the Credentials from Google Developer Console

7. Now we need add in the permissions that allow us to communicate with Google. To gain the proper credentials and their user database, we need to register and create an application with Google using the Developers API Console.

8.  If you have not created a project, create a new project by selecting "New Project" from your Dashboard.

9. After you have created a name for your Project, select "CREATE".

10. Select OAuth consent screen.

11. Fill in the Application name information and then select "Save".

12. Click on "Credentials" to create your credentials in order to access APIs.

13. Select "Create Credentials" and choose "OAuth client ID".

14. For our project purposes, we're creating a Web application. Since we're using the OAuth 2.0 protocol, Google generates an access token for our application, so select "Web Application".
            a) Fill in name for the Web Client
            b) The page requires a link for "Authorized JavaScript origins" and "Authorized redirect URIs". The origin URL is the URI in which requests from a  browser. The redirect URI is, from the Google page: "*For use with requests from a web server. This is the path in your application that users are redirected to after they have authenticated with Google. The path will be appended with the authorization code for access. Must have a  protocol. Cannot contain URL fragments or relative paths. Cannot be a public IP address.*"
                 For Authorized JavaScript origins, enter  since we are running our web application off of our local machine on port 3000 (using the rails server)
`        https://localhost:3000`
                 For Authorized redirect URIs, put the above URL as a placeholder. We will revisit this URL later.
            c) click "Create"
15. Google will now popup with a page that says "OAuth client". Copy the fields from "client ID" and "client secret"

### Setting Client Credentials to Environment Variables

16. Going back to your development area, in the root folder '/', create a ".env" file. In the file, enter the following:```
        GOOGLE_CLIENT_ID=<xxxxxx>
        GOOGLE_CLIENT_SECRET_KEY=<yyyyyy>
```

        xxxxxx is the "client ID"  and yyyyyy is the "client secret" that you copied from the steps above from the Google Developer Console.
17.  Make sure you add "*.env*" to your "*.gitignore*" file so that it doesn't mistakenly get committed and added to the repo. However, just remember that every time you pull a new clone from your github repository (or similar service), you need to re-create the .env file again.

## Configuring Devise to use the Correct Credentials

## Setting up the routes

18.  In "config/routes.rb" set up the routes required for devise by adding:

    devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
		
19. This lets Devise set the the OAuth2 callback to the omniauth_callback controller. We need to create  `app/controllers/users/omniauth_callbacks_controller.rb`. 
And then enter the following:
```
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController

    def google_oauth2
        @user = User.from_omniauth(request.env["omniauth.auth"])        puts @user.errors.to_a

        if @user.persisted?
            flash[:notice] = I18n.t 'devise.omniauth_callbacks.success', kind: 'Google'
            sign_in_and_redirect @user, :event => :authentication
        else
            session['devise.google_data'] = request.env['omniauth.auth'].except(:extra) # Removing extra as it can overflow some session stores
            redirect_to new_user_registration_url, alert: @user.errors.full_messages.join("\n")
        end   
		end
end
```
The action name needs to match the omniauth one is using. In our case, it is the google_oauth2, so our action name is called "google_oauth2"

20. In the OmniAuthsection in "`config/initializers/devise.rb`", add the following:
`config.omniauth :google_oauth2, ENV['GOOGLE_CLIENT_ID'], ENV['GOOGLE_CLIENT_SECRET_KEY'], {:name => "google_oauth2", scope: 'userinfo.email, userinfo.profile'}`

21. Also change config.scoped_views from false to true. This allows for custom views (the views under app/users/views/) instead of using devise's default views.`config.scoped_views = true`


22. Add the following to "app/controllers/application_controller.rb"
```
    protect_from_forgery prepend: true
    before_action :authenticate_user!   
```

23. Create "app/controllers/sessions_controller.rb"
```
class SessionsController < ApplicationController
    before_action :configure_permitted_parameters, if: :devise_controller?
    before_action :authenticate_user!
end
```

24. Find the authorization path. You can view all the routes by entering: `$rake routes`
25. Choose the prefix that includes the word "authorize" and "google_oauth2" in it. This is your google authorization path.
26. Modify the link in which to sign into Google under **app/views/user/shared/links.html**.Remove the following code:
```
          <%- if devise_mapping.omniauthable? %>
              <%- resource_class.omniauth_providers.each do |provider| %>
                   <%= link_to "Sign in with #{OmniAuth::Utils.camelize(provider)}", user_google_oauth2_omniauth_authorize_path(resource_name, provider) %><br />
                     <% end %>
          <% end %>
```
 and replace it with the route that is the authorization path. `<%= link_to "Sign in with Google", user_google_oauth2_omniauth_authorize_path %><br />`

26. Make sure to add to your root view  (the page that redirects the user to once they have properly logged in), a method for the user to log out.
`<%= link_to "LogOut",  destroy_user_session_path, method: :delete %>`

27. Find the callback path by entering:
`$rake routes`

28. Choose the prefix that includes the word "callback", user_google_oauth2_omniauth_callback in our example. Copy its URI Pattern, which is /users/auth/google_oauth2/callback for our project.

29.  Go back to the Developers Console. Select your project and click on "Credentials". Click on the pencil icon next to the name of your client which will edit its OAuth client. Append the URI Pattern to "https://localhost:3000", and add it as an entry. Remove the other entry. Example:
`https://localhost:3000/users/auth/google_oauth2/callback`

30. In your command console, enter:
`$thin start --ssl`

31. Navigate to [https://localhost:3000](https://localhost:3000) and now you should have your custom sign up form with a link to login with Google!


