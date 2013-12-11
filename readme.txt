This is an experiment to add invokedynamic support to Clojure.  The first
challenges were to upgrade the re-rooted ASM in Clojure to 4.0, and get Clojure
to generate V1.7 classfiles.  (I'm writing this in retrospect one year later, so
it may not be entirely accurate in all its details.)

Upgrading ASM was fairly simple, however, there were some changes that had to be
resolved.  For one, the ASM library changed from an interface based hierarchy to
a class inheritance based hierarchy.

Getting Clojure to generate v1.7 classfiles meant that the Clojure compiler was
now required to generate stackmaps.  ASM handles this for you automatically,
however there was an issue with the way it generated stackmaps that was causing
invalid classfiles.  The issue was resolved by extending the ASM ClassWriter's
getCommonSuperClass method to always return java.lang.Object (since Clojure is a
dynamic language that is about all we know at compile time).

Once those difficulties had been overcome it was time to attempt actually using
invokedynamic.  I chose to try something simple first.  I decided to try to
cache reflection information at callsites so that you would only have to pay the
cost of reflection once for the runtime of your application.

I was able to get something working, however, I believe that it is broken.  What
I implemented are monomorphic callsites, so for any use of reflection that is
polymorphic without any common super class or interface (there are is at least
one instance in the Clojure code base) I don't believe my callsites would work.
This would be an instance where you are calling a method that has the same name
on two different classes, but there is no common superclass through which you
can call that method on both of your targets.  Anyway, I don't know if this is a
practical concern; I'm just telling you.

Places to go next could be keyword callsites, protocol call sites, switchpoint
based vars, etc.

--------------------------------------------------------------------------

 *   Clojure
 *   Copyright (c) Rich Hickey. All rights reserved.
 *   The use and distribution terms for this software are covered by the
 *   Eclipse Public License 1.0 (http://opensource.org/licenses/eclipse-1.0.php)
 *   which can be found in the file epl-v10.html at the root of this distribution.
 *   By using this software in any fashion, you are agreeing to be bound by
 * 	 the terms of this license.
 *   You must not remove this notice, or any other, from this software.

Docs: http://clojure.org
Feedback: http://groups.google.com/group/clojure
Getting Started: http://dev.clojure.org/display/doc/Getting+Started

To run:  java -cp clojure-${VERSION}.jar clojure.main

To build locally with Ant:  

   One-time setup:    ./antsetup.sh
   To build:          ant

Maven 2 build instructions:

  To build:  mvn package 
  The built JARs will be in target/

  To build without testing:  mvn package -Dmaven.test.skip=true

  To build and install in local Maven repository:  mvn install

  To build a ZIP distribution:  mvn package -Pdistribution
  The built .zip will be in target/


--------------------------------------------------------------------------
This program uses the ASM bytecode engineering library which is distributed
with the following notice:

Copyright (c) 2000-2005 INRIA, France Telecom
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holders nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
THE POSSIBILITY OF SUCH DAMAGE.
