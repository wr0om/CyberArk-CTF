## Details
- Category: Innovation 
- Points: 178 (when I answered it)

## Description
```
You are orbiting mars, is it there where the intelligent life reside?
https://s3.us-west-2.amazonaws.com/cyber-ctf.be/mars.html
```
## Hint
```
(none)
```
## Solution
We are first sent to a good-looking website telling us they detected weird signals coming from mars.

When searching around on the site I found div with what looks like base64 encoded text:
```HTML
<div style="display: none;">
 ZXh0ZXJuIGNyYXRlIGJh...
</div>
```
After decoding the text in the div, I found out it was the language "Rust".
 <details>
    <summary><u>Here's the code</u> (<i>click to expand</i>):</summary>
  
```rust
extern crate base64;
use std::str;

// While exploring the surface of the red planet, you stumble upon an encrypted message, which might prove there's a life on this planet!
// After some sand digging, you found a rune, indicating the following:
// CMObw5jDlMOdw6JKK8OXw5TDocOUSg== | aliens? here?
// You are trying to build a cypher reader, but something is not quite right.
// What could this all mean?
//

trait IAbstractDecryptor {
    fn a(&self, msg: String) -> Vec<u8>;
    fn b(&self, b: Vec<u8>) -> String;
    fn c(&self, i: Vec<u8>) -> Vec<u8>;
    fn d(&self, by: Vec<u8>) -> String;
    fn e(&self, message: String) -> String;
}

trait IBaseDecryptor: IAbstractDecryptor {
    fn howl_loudly(&self);
}

struct Decryptor();

impl IAbstractDecryptor for Decryptor {
    fn a(&self, msg: String) -> Vec<u8>{
        return msg.into_bytes()

    }

    fn c(&self, i: Vec<u8>) -> Vec<u8> {
        return base64::decode(&i).unwrap();
    }

    fn d(&self, by: Vec<u8>) -> String{
        let c = self.c(by);
        let d = str::from_utf8(&c);
        return d.unwrap()
    }

    fn e(&self, message: String) -> String{
        let mut ss = String::from("");
        for c in message.chars() {
            let mut a = c as u32;
            let mut b = 1;
            let mut c = 0;
            
            while a > 0 {
                let mut d = a % 10;
                a /= 10;
                if d == 0 {
                    d = 10;
                }
                c = c + (d+1) * b;
                b *= 10;
            }
            if c < 10 {
                c += 290
            }

            ss.push(std::char::from_u32(c).unwrap())
        }
        return ss;
    }

    fn b(&self, b: Vec<u8>) -> String{
        let c = self.d(b);
        return self.e(c);
    }
}

impl IBaseDecryptor for Decryptor {
    fn howl_loudly(&self) {
        println!("on va le chercher toute la journée!!!!!!!!");
    }
}

fn main() {
    let _msg:String = String::from("YsOXCMOjKwgrw5vDnsOlw5TDm8OoK8OTCMOo");
    let decryptor = Decryptor();
    let a = decryptor.a(_msg);
    let extraterrestrial_msg = decryptor.b(a.clone());
}
```
</details>
  
To run and test this code I used the [Rust Playground](https://play.rust-lang.org/).
 
  We are met with comments at the start of the page:
  ```rust
  // While exploring the surface of the red planet, you stumble upon an encrypted message, which might prove there's a life on this planet!
// After some sand digging, you found a rune, indicating the following:
// CMObw5jDlMOdw6JKK8OXw5TDocOUSg== | aliens? here?
// You are trying to build a cypher reader, but something is not quite right.
// What could this all mean?
//
  ```
  This code contains two interfaces and a struct named Decryptor that implements both of them.
  The main function creates a string called "_msg" and a Decryptor struct. Let's study the code function by function:
  
  -> a(): This function simply takes a string and converts it into a vector of its ASCII decimal values.
  ```rust
  fn a(&self, msg: String) -> Vec<u8>{
        return msg.into_bytes()
    }
  ```
  -> c(): This function base64 decodes a vector and returns it.
  ```rust
fn c(&self, i: Vec<u8>) -> Vec<u8> {
        return base64::decode(&i).unwrap();
    }
  ```
  -> d(): This function base64 decodes a vector using c(), then converts it into a string and returns it. Notice I added .to_string() to the final line, this is required to return a string.
  ```rust
  fn d(&self, by: Vec<u8>) -> String{
        let c = self.c(by);
        let d = str::from_utf8(&c);
        return d.unwrap().to_string(); //added to help compile
    }
  ```
  -> e(): This function is long and a bit complicated, it does some manipulation of a string given, 1 char at a time - the logic itself is not so important.
  ```rust
  fn e(&self, message: String) -> String{
        let mut ss = String::from("");
        for c in message.chars() {
            let mut a = c as u32;
            let mut b = 1;
            let mut c = 0;
            
            while a > 0 {
                let mut d = a % 10;
                a /= 10;
                if d == 0 {
                    d = 10;
                }
                c = c + (d+1) * b;
                b *= 10;
            }
            if c < 10 {
                c += 290
            }

            ss.push(std::char::from_u32(c).unwrap())
        }
        return ss;
    }
  ```
 -> b(): This function base64 decodes a vector into a string using d(), returns e() of the string.
  ```rust
  fn b(&self, b: Vec<u8>) -> String{
        let c = self.d(b);
        return self.e(c);
    }
  ```
  -> main(): We get a string (_msg), a Decryptor, and the a() function is called on the string (turns the _msg into a decimal vector). Then, it calls the b() function with that vector.
  ```rust
  fn main() {
    let _msg:String = String::from("YsOXCMOjKwgrw5vDnsOlw5TDm8OoK8OTCMOo");
    let decryptor = Decryptor();
    let a = decryptor.a(_msg);
    let extraterrestrial_msg = decryptor.b(a.clone());
}
  ```
  To debug and print variables I used the <kbd> println! </kbd> command on some variables including the extraterrestrial_msg. The hint from the comments above said there was something wrong with the decryptor, and gave another subtle hint: "CMObw5jDlMOdw6JKK8OXw5TDocOUSg== | aliens? here?" - one example of decryption.
  
  From many tries and failures, I always got high decimal Unicode characters that didn't make any sense: "mņīŒ6ī6ŊōŔŃŊŗ6łīŗ". 
  After tons of tinkering and debugging, I decided to change one value on the e() function ->
  ```rust
          c = c + (d-1) * b;
  ```
  I changed the (d+1) to a (d-1), just because it was the only part that could lower the decimal value of the characters given, and when I put the string value from the hint (CMObw5jDlMOdw6JKK8OXw5TDocOUSg==) into the decoder, I got this: "ĩliens? here?".
  
  THIS WAS IT - I was so close. now with a little help of debugging, I found out that +290 was added to the first value - "ĩ" that should have been "a".
  The Unicode value of "ĩ" is 297, and the Unicode value of "a" is 97. I could then see it inside the code:
  ```rust
  if c < 10 {
     c += 90
  }
  ```
  As you can see, I changed the value of c+=290 to c+=90 and IT WORKED!. I got the value "aliens? here?" when decoding "CMObw5jDlMOdw6JKK8OXw5TDocOUSg==".
           </br>
  ## Flag
  Now, when decoding the string given in the main function, we get: "What a lovely day" - our flag
  
 
