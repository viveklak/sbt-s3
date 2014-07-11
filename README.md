# sbt-s3

## Description

*sbt-s3* is a simple sbt plugin that can manipulate objects on Amazon S3.

This is a fork of https://github.com/sbt/sbt-s3. This version allows specifying the S3 endpoint (read region) and bucket to
be specified explicitly instead of co-opting '''host''' - which didn't work for non-US standard regions.

Also tries to avoid downloading stuff when a local version already exists. Currently aborted the use of ETags to detect similarity
in favour of content length since some of my files are multi-part uploaded and the ETags seem to have different semantics for such
files. Will consider adding that in later.

## Usage

* add to your project/plugin.sbt the line:
   `addSbtPlugin("lakshmanan.ca.sbt" % "sbt-s3" % "0.8")`
* then add to your build.sbt the line:
   `s3Settings`
 
You will then be able to use the tasks s3-upload, s3-download, and s3-delete, defined
in the nested object `com.typesafe.sbt.S3Plugin.S3` as upload, download, and delete, respectively.
All these operations will use HTTPS as a transport protocol.
 
Please check the Scaladoc API of the `S3Plugin` object, and of its nested `S3` object,
to get additional documentation of the available sbt tasks.

## Example

Here is a complete example:

project/plugin.sbt:
    
    addSbtPlugin("lakshmanan.ca.sbt" % "sbt-s3" % "0.8")

build.sbt:

    import S3._

    s3Settings

    mappings in upload := Seq((new java.io.File("a"),"zipa.txt"),(new java.io.File("b"),"pongo/zipb.jar"))

    endpoint in upload := "s3-us-west-2.amazonaws.com"

    bucket in upload := "blah-blah" 
    credentials += Credentials(Path.userHome / ".s3credentials")

~/.s3credentials:

    realm=Amazon S3
    host=s3-us-west-2.amazonaws.com # You need this regardless, because sbt Credentials needs it :\
    user=<Access Key ID>
    password=<Secret Access Key>

Just create two sample files called "a" and "b" in the same directory that contains build.sbt, then try:

    $ sbt s3-upload
    
You can also see progress while uploading:

    $ sbt
    > set S3.progress in S3.upload := true
    > s3-upload
    [==================================================]   100%   zipa.txt
    [=====================================>            ]    74%   zipb.jar
