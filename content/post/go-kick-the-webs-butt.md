+++
date = "2015-10-31T15:11:05-05:00"
title = "Go kick the web's butt"
tags = ["github", "nodejs", "angular", "golang", "javascript", "martini"]
author = "Ivan Portugal"

deprecated = true

+++

## Intro
After getting introduced to [Go](https://golang.org/), I've fallen in love... Without making this corny, I really wanted to explore what the platform/language can do *immediately*. I'm sorta done with Hello World projects and going through examples of the language so I'd like to present a more-or-less real-world example.

Technologies used:
- Go
	- Martini
- Nodejs
- Grunt
- Bower

[Pseudo real-world Go Project](https://github.com/ivanbportugal/go-albums)

## Why Nodejs too?
I love web technologies. Nodejs has shaped how easy it is to develop projects for the web, including facilitating and supporting popular tools (via Github) like Grunt, Gulp, Bower, SASS, AngularJS, etc... Since I really like client-side development and I am a huge fan of AngularJS, I decided to make sure that whatever I came up with had this in mind.

Since Go runs and deploys very simply, it shouldn't be difficult to just incorporate it into a nice client-side workflow. So, here's a brief explanation of what I decided to do. I guess you could just click on the Github link above and not read on any further :(

## Martini
After wrestling a teeny bit with the `net/http` package, I decided to go with a nice router that was fairly-well accepted by the community. It seemed to provide nice facilities for auth too. Here's a snippet of the routing syntax:

```go
// Sets up reasonable defaults for your web app
var m = martini.Classic()

// CRUDL! Just pass in a callback
m.Get(`/albums`, server.GetAlbums)
m.Get(`/albums/:id`, server.GetAlbum)
m.Post(`/albums`, server.AddAlbum)
m.Put(`/albums/:id`, server.UpdateAlbum)
m.Delete(`/albums/:id`, server.DeleteAlbum)

// Inject database
m.MapTo(server.DBInstance, (*server.DB)(nil))
// Add the router action
m.Action(r.Handle)

// GetAlbums might look like this
func GetAlbums(r *http.Request, enc Encoder, db DB) string {
	// Get the query string arguments, if any
	qs := r.URL.Query()
	band, title, yrs := qs.Get("band"), qs.Get("title"), qs.Get("year")
	yri, err := strconv.Atoi(yrs)
	if err != nil {
		// If year is not a valid integer, ignore it
		yri = 0
	}
	if band != "" || title != "" || yri != 0 {
		// At least one filter, use Find()
		return Must(enc.Encode(toIface(db.Find(band, title, yri))...))
	}
	// Otherwise, return all albums
	return Must(enc.Encode(toIface(db.GetAll())...))
}
```

The important thing to note here is that the `GetAlbums` function takes in a `db` as a parameter because it's "injected" by the MapTo function. Perhaps Go has dependency injection out of the box that I haven't used but this feature is pretty useful, especially if I'd like to swap out for a different database in the future.

## Nodejs
Obviously `Go` does everything for the server so I just need a good client-side workflow. `Nodejs` is only used for development in this project. `Grunt` is used to configure different tasks as you might expect for minifying resources, launching live reload, etc... `Bower` manages clients-side dependencies. Examples of this are littered all over the web so I won't be discussing this except for the `Go` part.

Here's a `Gruntfile.js` snippet:
```javascript
	grunt.initConfig({
		goserver: {
            options: {
                // Port 3001 is HTTPS enforced
                port: 3000,
                // Change this to '0.0.0.0' to access the server from outside.
                hostname: 'localhost'
            },
            livereload: {
                options: {
                    staticDirs: ['.tmp', yeomanConfig.app]
                }
            },
            test: {
                options: {
                    port: 3002,
                    staticDirs: ['.tmp', 'test']
                }
            },
            dist: {
                options: {
                    port: 3003,
                    dist: '<%= gobuild.dist.options.dist %>',
                    staticDirs: [yeomanConfig.dist]
                }
            }
        }
	});

    function runGo(cmd, args, opts, done) {
        args.push('-p', opts.port);
        args.push('-h', opts.hostname);
        for (var i = 0; i < opts.staticDirs.length - 1; i++) {
            args.push('-static_dir', opts.staticDirs[i]);
        }
        if (opts.staticDirs.length > 0) {
            args.push(opts.staticDirs[opts.staticDirs.length - 1]);
        }

        var goProcess = grunt.util.spawn({
                cmd: cmd,
                args: args,
                opts: {
                    stdio: 'pipe'
                }
            },
            function(error, result, code) {
                if (error) {
                    grunt.log.error(String(result));
                    grunt.fail.fatal('go-server exited with code: ' + code, 3);
                }
            }
        );
        goProcess.stdout.pipe(process.stdout);
        goProcess.stderr.pipe(process.stderr);
        // Wait for spawned server to print something
        goProcess.stdout.once('data', function() {
            done();
        });
        process.on('exit', function() {
            grunt.log.writeln('Killing go-server(' + goProcess.pid + ')...');
            process.kill(-process.pid, 'SIGINT');
            grunt.log.oklns('Killed go-server');
        });

    }

    grunt.registerMultiTask('goserver', 'Running go server', function() {
        var opts = this.options({
            port: 9000,
            hostname: 'localhost',
        });
        if (opts.dist) {
            this.async(); // wait forever to keep the server alive
            runGo(opts.dist, [], opts, function() {});
        } else {
            runGo('go', ['run', 'main.go'], opts, this.async());
        }
    });

    grunt.registerMultiTask('gobuild', 'Building go server', function() {
        var opts = this.options({
            dist: './main',
            flags: [],
        });

        var done = this.async();
        grunt.util.spawn({
                cmd: 'go',
                args: ['build', '-o', opts.dist].concat(opts.flags),
                opts: {
                    stdio: 'inherit'
                }
            },
            function(error, result, code) {
                if (error) {
                    grunt.log.error(String(result));
                    grunt.fail.fatal('go-build exited with code: ' + code, 3);
                } else {
                    done();
                }
            }
        );
    });
```
Ok that was alot of code. Basically, it just ties `Go` executions and builds to the `Grunt` lifecycle. I had some help from other Github projects to put this together.

## AngularJS
Yeah, I'm an AngularJS (1.x) guy. I really could care less for server-side rendering for most apps because I feel that as browsers get better and better and devices get faster and faster, we can just let rendering be the responsibility of the client. I hate mudding the server-side waters sometimes, so my project is going to serve static resources and publish a HTTP RESTful API.

I suppose this might make it transparent when wanting to hook up different client apps built with/for iOS/Android, Chrome Web, [Electron](http://electron.atom.io/), [Thrust](https://github.com/miketheprogrammer/go-thrust) (new Go project), etc...

```javascript
    $http.get('/albums').then(function(data){
        // My albums are in JSON
        $scope.listings = data;
    }, function(err){
        // Some error happened
    });
```

## Conclusion
I guess if you're looking for cutting edge server-side web development I think you should `Go`, with confidence.

[Go Albums](https://github.com/ivanbportugal/go-albums)

### Kudos
- [Yeoman](http://yeoman.io/)
- [Angular Go Martini](https://github.com/rayokota/generator-angular-go-martini)
- [Martini API Example](https://github.com/PuerkitoBio/martini-api-example)