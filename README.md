# Maven-Release-Example

# Deploying to OSSRH with Apache Maven - Introduction

[Apache Maven](http://maven.apache.org/)  started the Central Repository by publishing all its components and required dependencies into the Central Repository - back then known as  _Maven Central_. Maven is pre-configured to connect to the Central Repository and download everything it needs including your project dependencies from the Central Repository.



# Other Prerequisites

Besides an Apache Maven installation, you have to have a GPG client installed and on your command line path as required by the Maven GPG plugin. For more information, please refer to  [http://www.gnupg.org/](http://www.gnupg.org/)  as well as the  [plugin documentation](http://maven.apache.org/plugins/maven-gpg-plugin/)  and below.


# Distribution Management and Authentication

In order to configure Maven to deploy to the OSSRH Nexus Repository Manager with the Nexus Staging Maven plugin you have to configure it like this

```
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
</distributionManagement>
<build>
  <plugins>
    <plugin>
      <groupId>org.sonatype.plugins</groupId>
      <artifactId>nexus-staging-maven-plugin</artifactId>
      <version>1.6.7</version>
      <extensions>true</extensions>
      <configuration>
        <serverId>ossrh</serverId>
        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
      </configuration>
    </plugin>
    ...
  </plugins>
</build>

```

Since OSSRH is always running the latest available version of Sonatype Nexus Repository Manager, it is best to use the latest version of the Nexus Staging Maven plugin.

Alternatively if you are using the Maven deploy plugin, which is the default behavior, you need to add a full  `distributionManagement`  section.

```
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
  <repository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
  </repository>
</distributionManagement>

```

The above configurations will get the user account details to deploy to OSSRH from your Maven  `settings.xml`  file. A minimal settings with the authentication is

```
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username>your-jira-id</username>
      <password>your-jira-pwd</password>
    </server>
  </servers>
</settings>

```

Note how the  `id`  element in the server element in  `settings.xml`  is identical to the  `id`  elements in the  `snapshotRepository`  and  `repository`element as well as the  `serverId`  configuration of the Nexus Staging Maven plugin

# Javadoc and Sources Attachments

To get Javadoc and Source jar files generated, you have to configure the javadoc and source Maven plugins.

```
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
      <version>2.2.1</version>
      <executions>
        <execution>
          <id>attach-sources</id>
          <goals>
            <goal>jar-no-fork</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-javadoc-plugin</artifactId>
      <version>2.9.1</version>
      <executions>
        <execution>
          <id>attach-javadocs</id>
          <goals>
            <goal>jar</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

# GPG Signed Components

The Maven GPG plugin is used to sign the components with the following configuration.

```
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-gpg-plugin</artifactId>
      <version>1.5</version>
      <executions>
        <execution>
          <id>sign-artifacts</id>
          <phase>verify</phase>
          <goals>
            <goal>sign</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

It relies on the gpg command being installed and the GPG credentials being available e.g. from settings.xml. In addition you can configure the gpg command in case it is different from gpg. This is a common scenario on some operating systems.

```
<settings>
  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.executable>gpg2</gpg.executable>
        <gpg.passphrase>the_pass_phrase</gpg.passphrase>
      </properties>
    </profile>
  </profiles>
</settings>
```

If you need more help setting up and configuring GPG, please read our  [detailed instructions](https://central.sonatype.org/pages/working-with-pgp-signatures.html).

# Nexus Staging Maven Plugin for Deployment and Release

The Nexus Staging Maven Plugin is the recommended way to deploy your components to OSSRH and release them to the Central Repository. To configure it simply add the plugin to your Maven pom.xml.

```
<build>
<plugins>
...
<plugin>
  <groupId>org.sonatype.plugins</groupId>
  <artifactId>nexus-staging-maven-plugin</artifactId>
  <version>1.6.7</version>
  <extensions>true</extensions>
  <configuration>
     <serverId>ossrh</serverId>
     <nexusUrl>https://oss.sonatype.org/</nexusUrl>
     <autoReleaseAfterClose>true</autoReleaseAfterClose>
  </configuration>
</plugin>
```

If your version is a release version (does not end in -SNAPSHOT) and with this setup in place, you can run a deployment to OSSRH and an automated release to the Central Repository with the usual:

```
mvn clean deploy
```

With the property autoReleaseAfterClose set to false you can manually inspect the staging repository in the Nexus Repository Manager and trigger a release of the staging repository later with

```
mvn nexus-staging:release
```

If you find something went wrong you can drop the staging repository with

```
mvn nexus-staging:drop
```

Please read  [Build Promotion with the Nexus Staging Suite](http://www.sonatype.com/books/nexus-book/reference/staging.html)  in the book  **Repository Management with Nexus**  for more information about the Nexus Staging Maven Plugin.


# Using a Profile

Since the generation of the javadoc and source jars as well as signing components with GPG is a fairly time consuming process, these executions are typically isolated from the normal build configuration and moved into a profile. This profile is then in turn used when a deployment is performed by activating the profile.

```
<profiles>
  <profile>
    <id>release</id>
    <build>
      ...
      javadoc, source and gpg plugin from above
      ...
    </build>
  </profile>
</profiles>

```

# Performing a Snapshot Deployment

Snapshot deployment are performed when your version ends in  `-SNAPSHOT`  . You do not need to fulfill the requirements when performing snapshot deployments and can simply run

```
mvn clean deploy
```

on your project.

SNAPSHOT versions are not synchronized to the Central Repository. If you wish your users to consume your SNAPSHOT versions, they would need to add the snapshot repository to their Nexus Repository Manager, settings.xml, or pom.xml. Successfully deployed SNAPSHOT versions will be found in https://oss.sonatype.org/content/repositories/snapshots/


# Additional Information

Maven release documnetation is very poor,  so make sure some steps while uploading your projetc to maven

- First rlease SNAPSHOT version that will be created locally and will be ready to go for release version make sure the version is -SNAPSHOT
like below example:
```
<groupId>Maven-Release-Example</groupId>
<artifactId>Maven-artifact-release</artifactId>
<version>1.0-SNAPSHOT</version>
```
- mvn clean deploy

after successfully uploading , remove the SNAPSHOT from the version. that should be like below:

```
<groupId>Maven-Release-Example</groupId>
<artifactId>Maven-artifact-release</artifactId>
<version>1.0</version>
```
- run again,  mvn clean deploy
this is responsible for creating the release version upload.

[In the case if you recieve error-]: *gpg: signing failed: Inappropriate ioctl for device*

You should set yout GPG_TTY variable for it to work, as in  [this document](https://www.gnupg.org/documentation/manuals/gnupg/Invoking-GPG_002dAGENT.html):

```
GPG_TTY=$(tty)
export GPG_TTY
```

# OpenPGP keyserver synchronize,

OpenPGP keyserver synchronize is very confusing so i am sharing my experience with you:

The various OpenPGP keyserver synchronize, but that takes some time. If you know which keyserver will be queried, you can directly upload your key there.

I did:

```
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys EE539F98
gpg --keyserver hkp://keyserver.ubuntu.com --send-keys EE539F98
```

and now your key can successfully be found on Ubuntu's keyserver, without having to wait until it automatically synchronized.

Actually I ran the recv-command multiple times to find a keyserver in their pool which already had your key.

- Note: while releasing SNAPSHOT version there will profile is will be generated and it will looks like

```

Last login: Tue Feb 26 12:59:14 on console

Shaileshs-MacBook-Pro:~ shaileshmishra$ $ gpg --full-generate-key
-bash: $: command not found
Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --list-secret-keys
gpg: keybox '/Users/shaileshmishra/.gnupg/pubring.kbx' created
gpg: /Users/shaileshmishra/.gnupg/trustdb.gpg: trustdb created
Shaileshs-MacBook-Pro:~ shaileshmishra$ $ gpg --full-generate-key
-bash: $: command not found
Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --full-generate-key
gpg (GnuPG) 2.2.11; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:

(1) RSA and RSA (default)
(2) DSA and Elgamal
(3) DSA (sign only)
(4) RSA (sign only)

Your selection? 1

RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.

0 = key does not expire
<n>  = key expires in n days
<n>w = key expires in n weeks
<n>m = key expires in n months
<n>y = key expires in n years
Key is valid for? (0) 2y

Key expires at Thu Feb 25 13:41:52 2021 IST
Is this correct? (y/N) y
GnuPG needs to construct a user ID to identify your key.

Real name: Maven-Release-Example
Email address: mshailesh@gmail.com
Comment: maven version released

You selected this USER-ID:

"Maven-Release-Example (maven version released) <mshailesh@gmail.com>"



Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 94BAQOECA23232133 marked as ultimately trusted
gpg: directory '/Users/shaileshmishra/.gnupg/openpgp-revocs.d' created

gpg: revocation certificate stored as '/Users/shaileshmishra/.gnupg/openpgp-revocs.473298478392743874478234836487236475354.rev'

public and secret key created and signed.
pub rsa2048 2019-02-26 [SC] [expires: 2021-02-25]
473298478392743874478234836487236475354
uid  Maven-Release-Example (maven version released) <mshaileshr@gmail.com>
sub rsa2048 2019-02-26 [E] [expires: 2021-02-25]

Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --list-secret-keys --94BAQOECA23232133-format LONG
gpg: invalid option "--94BAQOECA23232133-format"
Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --list-secret-keys --94BAQOECA23232133
gpg: invalid option "--94BAQOECA23232133"
Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --list-secret-keys 94BAQOECA23232133

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid: 1  signed: 0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2021-02-25
sec rsa2048 2019-02-26 [SC] [expires: 2021-02-25]
473298478392743874478234836487236475354

uid [ultimate] Maven-Release-Example (maven version released) <mshaileshr@gmail.com>

ssb rsa2048 2019-02-26 [E] [expires: 2021-02-25]


- Open your github account's settings select pgp section and put pgp String in container.
Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --armor --export 94BAQOECA23232133

-----BEGIN PGP PUBLIC KEY BLOCK-----
mQENBFx09O8BCAC0PKDhP0vLLpg/OzN+weriunfiunerifnirefnierunfiuen
ifirnfierunfinerifnernffjiuerhfirniubnrtgiuriugirturjnjernfkjwe
HgBnBW8IwwRSI2QLbArnpKOZ4u4x3ru5LWwxXvi1eFTIQ+zKdM67rvd8jzmbyio+
prKigmIKR5EOgyOYUv97/u4UYNt8zN8FOKc0SGKK/sdAGL9EXu/8fAlO48dW8zKy
0Wyc99ZkNTDIRZ91N6U+WAH6yT0BhpkKr/+RFPkmDA/YqYA/w2tM26NH5qNsCILV
up2yyvt3T6+7/kjnwednwendiuewndieuwndiuenwiudfneiudfniendfiuendi
ayAobWF2ZW4gdmVyc2lvbiByZWxlYXNlZCkgPG1vYmlsZUBjb250ZW50c3RhY2su
Y29tPokBVAQTAQgAPhYhBCPTgsNaPdfyUNSE25Q6K+yuCxJCBQJcdPTvAhsDBQkD
mGEEgyVrFOdw0ry9xR1erW4urMOGJhtyWZmJHs3e2R9GJSYciiLONwAGniZra2lk
w7oEJbaFXmpHKfPM9ttkKlS8IRpULYbiTxIZ7E6KqiJ4lOM7UdqGgxAenedJLQna
qkN9SD/tm6PR6uUP2GRyY1mhmp2XZ7oy2517QipiUllgm651x+QiFfi4aKfwAjfd
Y2W+gR4IYBrCyi7eV3rs16ZLT89ogsaD3hmgvLiOFpQylsv6ns4UPEXVmeOH3Xx1
mGEEgyVrFOdw0ry9xR1erW4urMOGJhtyWZmJHs3e2R9GJSYciiLONwAGniZra2lk
nZaRso4SZHjdpxzOPgh7pA/TdyX9lnWnAHTQAQW4exaKII+9CavoAlK/2cYlrae
l3QPIlEWhZthDHW5AQ0EXHT07wEIANe1snCJc6bdW6gFvVgswd1r4Vr858n8oYgh
2t4ukfcgvZsjN2OYEilxTTl3avZ+kiPSELif3nYmc3nq0ZQD34Oh79sVVj4plT+g
mGEEgyVrFOdw0ry9xR1erW4urMOGJhtyWZmJHs3e2R9GJSYciiLONwAGniZra2lk
Y2W+gR4IYBrCyi7eV3rs16ZLT89ogsaD3hmgvLiOFpQylsv6ns4UPEXVmeOH3Xx1
mGEEgyVrFOdw0ry9xR1erW4urMOGJhtyWZmJHs3e2R9GJSYciiLONwAGniZra2lk
IOZE30bbYnpJUd9lHd0ayj3lXVGJDLlBDgIkvPRfvfgkeCJWYJ0AEQEAAYkBPAQY
AQgAJhYhBCPTgsNaPdfyUNSE25Q6K+yuCxJCBQJcdPTvAhsMBQkDwmcAAAoJEJQ6
K+yuCxJCE7IIAI3gcs2gIyb1PSuZ+MseyS0m9el6yyNmFmUziZ/+4WpPwCdq4rJ+
xYTfmiTB/zHe7SjcioB67dG3Tr1LTFKrjx1Bo7wDApWe2hNjffu2QTzGG71SXrWZ
dIcr7taC4licaNNqythYVDe/sgis5magKVxdeOGah+BKg9Gm+eJMat/QPgjfRN70
mGEEgyVrFOdw0ry9xR1erW4urMOGJhtyWZmJHs3e2R9GJSYciiLONwAGniZra2lk
8R7rRlXWIGJe11x4o8te93fVW/9xFN1BzuhPyximG6fr+zZFYLW2fFscMbvWP/5G
EGPMpNqDAE2pE3HMTvEbzA2SYPJRz3op/iY==52VU
-----END PGP PUBLIC KEY BLOCK-----

make sure you run recv commond first and then send keys:
```
gpg --keyserver hkp://pool.sks-keyservers.net --recv-keys 94BAQOECA23232133
gpg --keyserver hkp://keyserver.ubuntu.com --send-keys 94BAQOECA23232133
```


Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --keyserver hkp://pgp.mit.edu --recv-keys 94BAQOECA23232133
gpg: sending key 94BAQOECA23232133 to hkp://pgp.mit.edu
Shaileshs-MacBook-Pro:~ shaileshmishra$ gpg --keyserver hkp://pgp.mit.edu --send-keys 94BAQOECA23232133
```


That's IT. Enjoy


Thanks
Shailesh Mishra