### Playground

The easiest way to try out these examples is by downloading the [Python Playground]() app. This app allows you to paste code into a web based editor and run it as if it were built as part of the app!

Once downloaded, start the app, and then go to [http://your-phone-address:8888](http://localhost:8888). If using a simulator run `adb forward tcp:8888 tcp:8888` and go to [http://localhost:8888](http://localhost:8888).

Copy and paste the example code in and click the play button. The app reloads and there you go! You can try out any code this way as well so feel free to play around!

### Basics

#### Text

Use a `TextView` to show text. You can set color, size, font, and other properties.  
    :::python
    from enamlnative.widgets.api import *

    enamldef ContentView(Flexbox):
        TextView:
            text = "Hello world!"
            text_color = "#00FF00"
            text_size = 32
            font_family = "sans-serif"



#### Text Inputs
You can observe text input changes by binding to the `text` attribute.

    :::python
    from enamlnative.widgets.api import *

    enamldef ContentView(Flexbox):
        flex_direction = "column"
        EditText: et1:
           pass
        EditText: et2:
           #: Two way binding
           text := et1.text
        TextView:
            text << "You typed: {}".format(et1.text)




#### Toggle Switch 
You can handle Switch, CheckBox, and ToggleButton checked changes with the `checked` attribute.

    :::python
    from enamlnative.widgets.api import *
    
    enamldef ContentView(Flexbox):
        flex_direction = "column"
        Switch: sw:
           text = "Switch"
        CheckBox: cb:
            text = "Checkbox"
            #: Two way binding
            checked := sw.checked
        PushButton:
            text = "PushButton"
            checked := sw.checked
        TextView:
            text << "Switch state: {}".format(sw.checked)



#### Buttons 
You can handle button clicks with the `clicked` event.

    :::python
    from enamlnative.widgets.api import *

    enamldef ContentView(Flexbox):
        Button: btn:
           attr clicks = 0
           text = "Click me!"
           clicked :: self.clicks +=1
        TextView: txt:
           text << "Clicked: {}".format(btn.clicks)



#### Clickable 
Any `View` can be made clickable by setting `clickable=True` and using the `clicked` event.

    :::python
    from enamlnative.widgets.api import *

    enamldef ContentView(Flexbox):
        TextView: txt:
           attr clicks = 0
           text << "Click me: {}".format(self.clicks)
           clickable = True
           clicked :: self.clicks +=1



#### Width and Height 
You can define size using `layout_width` and `layout_height`. Can be either an integer string `200` (in dp), `wrap_content`, or `match_parent`.

    :::python
    from enamlnative.widgets.api import *

    enamldef ContentView(Flexbox):
        orientation = "vertical"
        Flexbox: 
           background_color = '#00FF00'   
           layout = dict(flex_basis = 0.3)
        Flexbox: 
           background_color = '#FFFF00'   
           layout = dict(flex_basis = 0.2)
        Flexbox: 
           background_color = '#0000FF'  
           layout = dict(flex_basis = 0.5)




### App Login

An app that shows simple login and logout screens. 

[![See the demo on youtube](https://img.youtube.com/vi/mSb37wTMfW0/0.jpg)](https://youtu.be/mSb37wTMfW0)

    :::python

    """

    A simple login app example.


    """
    from atom.api import *
    from enaml.core.api import *
    from enamlnative.widgets.api import *
    from enamlnative.core.app import BridgedApplication

    class User(Atom):
        username = Unicode()
        password = Unicode()


    class App(Atom):
        """ App state and controller """

        theme_color = Unicode("#cab")

        #: Define our "user database"
        users = List(User,default=[
            User(username="Bob",password="secret"),
            User(username="Jane",password="sweet"),
        ])

        #: Current user
        current_user = Instance(User)

        def login(self,user,password):
            """ Return a user if the user and pwd match """
            #: Simulate a login that takes time
            app = BridgedApplication.instance()

            #: Get an async result
            result = app.create_future()

            def simulate_login(result, user,password):
                for u in self.users:
                    if u.username==user and u.password==password:
                        self.current_user = u
                        result.set_result(self.current_user)
                        return

                #: No passwords match!
                self.current_user = None
                result.set_result(self.current_user)

            #: Simulate the call taking some time
            #: In a real app we would check using some web service
            app.timed_call(1000,simulate_login,result,user,password)

            return result



    enamldef SignInScreen(PagerFragment): view:
        attr app: App
        attr working = False
        attr pager << view.parent
        attr error = ""
        Flexbox:
            flex_direction = "column"
            background_color = "#eee"
            justify_content = "center"
            padding = (30, 30, 30, 30)

            Flexbox:
                justify_content="center"
                Flexbox:
                    layout_height="wrap_content"
                    layout_width="wrap_content"
                    flex_direction="column"
                    justify_content="center"
                    #background_color="#f00"
                    Icon:
                        text = "{fa-rocket}"
                        text_size = 128
                        text_color << view.app.theme_color
                    TextView:
                        text = "Your company"
            Flexbox:
                flex_direction = "column"
                #layout = dict(flex_basis=0.4)
                TextView:
                    text = "Username"
                EditText: username:
                    #: So it clears when the user is reset
                    text << "" if view.app.current_user else ""
                TextView:
                    text = "Password"
                EditText: password:
                    #: So it clears when the user is reset
                    text << "" if view.app.current_user else ""
                    input_type = "text_web_password"

                TextView:
                    text << view.error
                    text_color = "#f00"
                Conditional: cond:
                    condition << bool(view.working)
                    ActivityIndicator:
                        padding = (0,10,0,0)
                        style="small"                    
                Conditional:
                    condition << bool(not view.working and not view.app.current_user)
                    Button:
                        style = "borderless"
                        text << "Sign In"
                        #text_color << view.app.theme_color
                        attr root << view
                        func on_login_result(r):
                            #: Why is scope screwed up?
                            view = self.root
                            view.working = False
                            if r is None:
                                view.error = "Invalid username or password"
                            else:
                                view.error = ""
                                view.pager.current_index +=1

                        clicked :: 
                            view.working = True
                            self.root = view
                            #: Simulate an async login request
                            view.app.login(
                                username.text,
                                password.text).then(on_login_result)

                Conditional:
                    #: Dispal
                    condition << view.app.current_user is not None
                    Flexbox:
                        layout_height = "wrap_content"
                        justify_content = "center"
                        Icon:
                            padding = (0,10,0,0)
                            text = "{fa-check}"
                            text_size = 32
                            text_color << view.app.theme_color


    enamldef HomeScreen(PagerFragment): view:
        attr app: App
        attr user << app.current_user
        Flexbox:
            background_color << "#eee"
            justify_content = "center"
            Flexbox:
                flex_direction="column"
                Flexbox:
                    justify_content = "center"
                    Icon:
                        padding = (0,10,0,0)
                        text = "{fa-thumbs-up}"
                        text_size = 128
                        text_color << view.app.theme_color
                Flexbox:
                    justify_content = "center"
                    TextView:
                        text << "{}, you rock!".format(view.user.username) if view.user else ""
                        text_color << view.app.theme_color
                Flexbox:
                    flex_direction = "column"
                    justify_content = "flex_end"
                    Button:
                        style = "borderless"
                        text = "Logout"
                        clicked :: 
                            view.app.current_user = None
                            view.parent.current_index = 0


    enamldef ContentView(Flexbox): root:
        #: Our app state
        attr app = App()
        ViewPager: 
            #: Don't let them go by swiping!
            paging_enabled = False
            SignInScreen:
                app << root.app
            HomeScreen:
                app << root.app

### App Intro

A simple app intro screen with dots for paging and next/back buttons. 

[![See the demo on youtube](https://img.youtube.com/vi/UxctC4L2zD0/0.jpg)](https://youtu.be/UxctC4L2zD0)

    :::python
    """

    A simple app intro screen!

    """
    from enamlnative.core.api import *
    from enamlnative.widgets.api import *

    enamldef PagerDots(Flexbox):
        attr icon = "fa-circle"
        attr color = "#fff"
        attr pager
        layout_height = "100"
        justify_content = "space_between"
        align_content = "center"
        layout = dict(align_self = "flex_end")
        attr pages << [c for c in pager._children if isinstance(c,Fragment)]
        attr next_enabled = True
        attr back_enabled = True
        Flexbox:
            Button:
                enabled << pager.current_index>0 and back_enabled
                style = "borderless"
                text << "Back" if self.enabled else ""
                text_color << color
                clicked :: pager.current_index -=1
        Flexbox:
            #layout_width = "wrap_content"
            justify_content = "center" 
            align_items = "center"
            Looper:
                iterable << range(len(pages))
                Icon:
                    text = "{%s}"%icon
                    padding = (5,5,5,5)
                    text_color << color
                    alpha << 1 if pager.current_index==loop_index else 0.4
                    clickable = True
                    clicked :: pager.current_index = loop_index
        Flexbox:
            Button:
                enabled << pager.current_index+1<len(pages) and next_enabled
                style = "borderless"
                text << "Next" if self.enabled else ""
                text_color << color
                clicked :: pager.current_index +=1

    enamldef AppIntro(Flexbox): view:
        alias screens
        flex_direction = "column"
        #padding = (10, 10, 10, 10)
        ViewPager: view_pager:
            Block: screens:
                pass
        PagerDots: dots:
            pager << view_pager

    enamldef Text(TextView):
      text_color = "#fff"
      text_size = 18
      font_family = "casual"

    enamldef HomeScreen(PagerFragment):
        Flexbox:
            flex_direction = "column"
            padding = (10,10,10,10)
            ImageView:
                src = "@mipmap/ic_launcher"
            Text:
              text = "Welcome to the python playground!"
              text_size = 32
            Text:
              padding = (0, 30, 0, 0)
              text = "This app lets you write an Android app using python from your web browser!!"
            Text:
              padding = (0, 30, 0, 0)
              text = "Swipe or click next to get started."

    enamldef GettingStartedScreen(PagerFragment):
        Flexbox:
            flex_direction = "column"
            padding = (10,10,10,10)
            Flexbox:
                flex_direction = "column"
                layout = dict(flex_basis=0.7)
                Text:
                  text = "Getting Started"
                  text_size = 32
                Text:
                  padding = (0, 30, 0, 0)
                  text = "Open settings and get the Wifi IP address of your device. " \
                         "Now open your browser and go to:"
                Flexbox:
                    layout_height = 'wrap_content'
                    justify_content = "center"
                    Text:
                        text = "http://<device-ip>:8888/"
                Text:
                  padding = (0, 30, 0, 0)
                  text = "If using a simulator, run:"
                Flexbox:
                    layout_height = 'wrap_content'
                    justify_content = "center"      
                    Text:
                        text = "adb forward tcp:8888 tcp:8888"
                Text:
                     text = "and go then to:"
                Flexbox:
                    layout_height = 'wrap_content'
                    justify_content = "center"
                    Text:
                        text = "http://localhost:8888/"
            Flexbox:
                justify_content = "center"
                align_items = "center"
                layout = dict(flex_basis=0.3)
                Icon:
                  text = "{fa-terminal}"
                  text_size = 128

    enamldef PlayScreen(PagerFragment):
        Flexbox:
            flex_direction = "column"
            padding = (10,10,10,10)
            Flexbox:
                flex_direction = "column"
                layout = dict(flex_basis=0.7)
                Text:
                  text = "Play!"
                  text_size = 32
                Text:
                  padding = (0, 30, 0, 0)
                  text = "Enter your code in the editor and press play! "\
                         "The app will reload with your code!"
                Text:
                  padding = (0, 30, 0, 0)
                  text = "Documentation and examples can be found at: "
                Text:
                    text_color = "#123"
                    text = "www.codelv.com/projects/enaml-native/docs/"
            Flexbox:
                justify_content = "center"
                align_items = "center"
                layout = dict(flex_basis=0.3)
                Icon:
                  text = "{fa-rocket}"
                  text_size = 128


    enamldef ContentView(AppIntro): view:
      background_color = "#6CA6CD"
      Block:
        block = parent.screens
        HomeScreen:
            pass
        GettingStartedScreen:
            pass
        PlayScreen:
            pass






