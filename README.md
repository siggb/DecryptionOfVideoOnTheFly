## Decryption Of Video On The Fly

We have an encrypted video file (e.g., using XOR function: `XOR(byte_from_file, byte_from_key)`). The goal is to decrypt the video stream on the fly.

We'll use a built-in simple HTTP [web-server](https://github.com/swisspol/GCDWebServer) in our app. The app receives the encrypted stream, decrypts it, and streams it again through the built-in HTTP server.

### Requirements

* Xcode 5 or higher
* Apple LLVM compiler
* iOS 6.0 or higher
* ARC

### Example Usage

Initializing the web server [here](https://github.com/siggb/DecryptionOfVideoOnFly/blob/master/Sources/Pages/Main/MainViewController.m):
```objective-c
webServer = [[GCDWebServer alloc] init];
[webServer addDefaultHandlerForMethod:@"GET"
                         requestClass:[GCDWebServerRequest class]
                         processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
                             DPLogFast(@"request : %@", request);
                             GCDWebServerFileResponseWithDecryption *response = [GCDWebServerFileResponseWithDecryption responseWithFile:[[NSBundle mainBundle] pathForResource:@"video" ofType:@"mov"]
                                                                                                                           byteRange:[request byteRange]
                                                                                                                        isAttachment:NO];
                             DPLogFast(@"response : %@", response);
                             return response;
                         }];
```

Redefinition of the read-data method [here](https://github.com/siggb/DecryptionOfVideoOnFly/blob/master/Sources/Customs/GCDWebServerFileResponseWithDecryption.m):
```objective-c
- (NSData*)readData:(NSError**)error
{
    NSData* data = [super readData:error];
    
    // Decoding part
    NSData *key_data = [@"KEYKEYKEYKEYKEYKEYKEYKEYKEY" dataUsingEncoding:NSUTF8StringEncoding];
    uint8_t *bytes = (uint8_t *)data.bytes;
    uint8_t *key_bytes = (uint8_t *)key_data.bytes;
    
    NSMutableData *m_file_data = [NSMutableData dataWithCapacity:data.length];
    for (int i = 0, j = 0; i < data.length; i++, j++) {
        if (j >= key_data.length) j = 0;
        uint8_t m_byte = bytes[i] ^ key_bytes[j];
        [m_file_data appendBytes:&m_byte length:1];
    }
    
    return ARC_AUTORELEASE(m_file_data);
}
```

### License

Decryption Of Video On Fly is available under the MIT license.

```
MIT License

© 2013 Ildar Sibagatov

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```
