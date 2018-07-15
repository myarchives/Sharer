Sharer
======

### A personal and configurable file/link sharer

## Deploy
This app uses [Google App Engine](https://cloud.google.com/appengine/) as the underlying system. The Google App Engine free tier should suffice for personal usage.
1. Install the gcloud sdk
    - https://cloud.google.com/sdk/install
2. Login to gcloud
    - `gcloud auth login`
3. Update your `app.yaml` to reflect your configuration
4. Run the gcloud app deploy on the project for Sharer
    - `gcloud app deploy app.yaml`
    
## Usage
I personally live in the terminal, so it has been made to be used terminal first. Maybe one day I'll add a Web UI, but this works just fine.
Here are the functions that I use in ZSH. Your mileage may vary:

These snippets have been adapted from ones provided by [Dutchcoders](https://dutchcoders.io) for [transfer.sh](https://transfer.sh):
```bash
share() { 
    # check arguments
    if [ $# -eq 0 ]; 
    then 
        echo "No arguments specified. Usage:\necho share /tmp/test.md 10m 10 #(clicks)\ncat /tmp/test.md | share test.md 10m 10 #(clicks)"
        return 1
    fi

    # get temporarily filename, output is written to this file show progress can be showed
    tmpfile=$( mktemp -t transferXXX )
    
    # upload stdin or file
    file=$1

    if tty -s; 
    then 
        basefile=$(basename "$file" | sed -e 's/[^a-zA-Z0-9._-]/-/g') 

        if [ ! -e $file ];
        then
            echo "File $file doesn't exists."
            return 1
        fi
        
        if [ -d $file ];
        then
            # zip directory and transfer
            zipfile=$( mktemp -t transferXXX.zip )
            cd $(dirname $file) && zip -r -q - $(basename $file) >> $zipfile
            curl -H "X-Authorization: AUTH_TOKEN" --progress-bar --upload-file "$zipfile" "https://HOSTNAME/api/upload/$basefile.zip?s=1&time=$2&clicks=$3" >> $tmpfile
            rm -f $zipfile
        else
            # transfer file
            curl -H "X-Authorization: AUTH_TOKEN" --progress-bar --upload-file "$file" "https://HOSTNAME/api/upload/$basefile?s=1&time=$2&clicks=$3" >> $tmpfile
        fi
    else 
        # transfer pipe
        curl -H "X-Authorization: AUTH_TOKEN" --progress-bar --upload-file "-" "https://HOSTNAME/api/upload/$file?s=1&time=$2&clicks=$3" >> $tmpfile
    fi
   
    # cat output link
    cat $tmpfile

    # cleanup
    rm -f $tmpfile
}
```

```bash
linkshare() { 
    # check arguments
    if [ $# -eq 0 ]; 
    then 
        echo "No arguments specified. Usage:\necho linkshare https://google.com 10m 10 #(clicks)"
        return 1
    fi

    if tty -s; 
    then
        curl -H "X-Authorization: AUTH_TOKEN" -X "POST" "https://HOSTNAME/api/shorten?s=1&url=$1&time=$2&clicks=$3"
    fi
}
```

Update your own hostname (`s/HOSTNAME/YOUR_HOST_HERE/g`) and auth key `s/AUTH_TOKEN/YOUR_AUTH_KEY/g`

Example usages are as follows:

```
# share <file> <duration> <click count>
share superman.jpg 10m 2
```

```
# linkshare <url> <duration> <click count>
linkshare https://google.com 10m 2
```