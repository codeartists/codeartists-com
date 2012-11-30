## How to directly upload files to Amazon S3 from your client side web app

### Why you need this?

You don't want your heavy-weight data to travel 2 legs from Client to Server to S3, incurring the cost of IO 
and clogging the pipe 2 times.

Instead, you want to ask your server to give your client one-time permission to upload your data
directly to S3. The process is still 2 legged, but heawy-weight data travels only on 1 leg.

On Amazon S3 this is implemented with CORS (Cross Origin Resource Sharing)

We use this at Dubjoy, where customers upload their huge video files to S3 for translation and 
voice-over, and we don't want our Heroku server to have anything to do with heavy-weight video files.

### Steps to implement this

1. Set up Amazon S3 bucket CORS configuration
2. Implement client-side JavaScript (CoffeScript, JavaScript)
3. Implement server-side upload request signing (Ruby/Sinatra, trivial to do in any other language)

### 1. Amazon S3 bucket CORS configuration

Set this in AWS S3 management console. Right-click on the desired bucket and select Properties. 
Below, on the permissions tab, click Edit CORS configuration, paste the XML below and click Save.

    <?xml version="1.0" encoding="UTF-8"?>
    <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>GET</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>Authorization</AllowedHeader>
        </CORSRule>
        <CORSRule>
            <AllowedOrigin>*</AllowedOrigin>
            <AllowedMethod>PUT</AllowedMethod>
            <MaxAgeSeconds>3000</MaxAgeSeconds>
            <AllowedHeader>Content-Type</AllowedHeader>
            <AllowedHeader>x-amz-acl</AllowedHeader>
            <AllowedHeader>origin</AllowedHeader>
        </CORSRule>
    </CORSConfiguration>

### 2. Your web app

Head to [GitHub](https://github.com/tadruj/s3upload-coffee-javascript) repo with CoffeScript and JavaScript Class files to include.
In your app, do the following:

#### HAML

    `%input#file{ :type => 'file', :name => 'files[]'}`

#### or HTML for the chevron-lovers

    `<input type='file' name='files[]' />`

#### CoffeScript

    s3upload = s3upload ? new S3Upload
    	file_dom_selector: '#files'
      s3_sign_put_url: '/signS3put'
    	onProgress: (percent, message) ->
    		console.log 'Upload progress: ', percent, message # Use this for live upload progress bars
    	onFinishS3Put: (public_url) ->
    		console.log 'Upload finished: ', public_url # Get the URL of the uploaded file
      onError: (status) ->
        console.log 'Upload error: ', status

#### or JavaScript for the brace-lovers

    var s3upload = s3upload != null ? s3upload : new S3Upload({
      file_dom_selector: '#files',
      s3_sign_put_url: '/signS3put',
      onProgress: function(percent, message) { // Use this for live upload progress bars
        console.log('Upload progress: ', percent, message);
      },
      onFinishS3Put: function(public_url) { // Get the URL of the uploaded file
        console.log('Upload finished: ', public_url);
      },
      onError: function(status) {
        console.log('Upload error: ', status);
      }
    });

Be sure to set the right DOM selector name `file_dom_selector` for file input tag, #files in our case. 
`s3_sign_put_url` is an end-point on your server where you will be signing S3 PUT requests.

### 3. Server-side request signing

Be sure to set `S3_BUCKET_NAME`,`S3_SECRET_KEY`,`S3_ACCESS_KEY`. 
Create a bucket and get the keys under Security Credentials menu in AWS management console.

    S3_BUCKET_NAME = 'CREATE_A_BUCKET_AND_SET_THE_NAME_HERE'
    S3_SECRET_KEY = 'GET_THIS_IN_AWS_CONSOLE'
    S3_ACCESS_KEY = 'GET_THIS_IN_AWS_CONSOLE'

    get '/signS3put' do
      objectName = params[:s3_object_name]
      mimeType = params['s3_object_type']
      expires = Time.now.to_i + 100 # PUT request to S3 must start within 100 seconds

      amzHeaders = "x-amz-acl:public-read" # set the public read permission on the uploaded file
      stringToSign = "PUT\n\n#{mimeType}\n#{expires}\n#{amzHeaders}\n/#{S3_BUCKET_NAME}/#{objectName}";
      sig = CGI::escape(Base64.strict_encode64(OpenSSL::HMAC.digest('sha1', S3_SECRET_KEY, stringToSign)))

      {
        signed_request: CGI::escape("#{S3_URL}#{S3_BUCKET_NAME}/#{objectName}?AWSAccessKeyId=#{S3_ACCESS_KEY}&Expires=#{expires}&Signature=#{sig}"),
        url: "http://s3.amazonaws.com/#{S3_BUCKET_NAME}/#{objectName}"
      }.to_json
    end


The code is a based on these resources, but has been put in to an easy to use CoffeeScript/JavaScript Class

* http://docs.amazonwebservices.com/AmazonS3/latest/dev/cors.html#how-do-i-enable-cors
* http://www.ioncannon.net/programming/1539/direct-browser-uploading-amazon-s3-cors-fileapi-xhr2-and-signed-puts/
* https://github.com/carsonmcdonald/direct-browser-s3-upload-example

You can learn more about CORS here

* http://www.html5rocks.com/en/tutorials/cors/
* http://remysharp.com/2011/04/21/getting-cors-working/
