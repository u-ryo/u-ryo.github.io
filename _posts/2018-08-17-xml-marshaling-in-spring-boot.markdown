---
layout: post
title: "XML marshaling in Spring Boot"
date: "2018-08-17 20:33"
author: 'u-ryo'
categories: [java, spring, spring boot, xml, jaxb, jackson]
comments: true
published: true
---
Spring Boot applicationで単一xml fileを返すREST作ってあって、
そこではxmlと同じ構造のJava bean作って返すだけで、
marshalingについてはframework側がよしなにやってくれました。
client側がHTTP Request Headerに`Accept: application/xml`とすれば。
そうでないとjsonになります。まぁそれはそれでいいんです。いいと思いました。

その後、複数xmlをまとめてZIPにして返すRESTを求められました。
そうすると、自分でXMLにmarshalingしなければなりません。
と、Gson?
でも`build.gradle`見ると折角jackson読み込んでいるようなので、
jacksonでmarshalしました。`new ObjectMapper()`して、
`mapper.write(ZipOutputStream)`みたいなことすると、
一回(=one file)書いただけでstreamを勝手に? closeするようなので、
一旦`String`にしてから`zos.write()`しました。
また、`build.gradle`に`compile "...jackson-dataformat-xml"`も必要でした。

しかしそうすると、今度はXMLを返すRESTの方で、
返されるXMLの形が微妙に違っていました。具体的には、

1. `@XmlRootElement(name=...)`で指定した名前が効かない
1. XML Object中で`List`要素がnestされる
(`<object></object><object></object>...`だったのが
`<object><object></object><object></object>...</object>`に)

`@XmlRootElement`はJAXBのannotation
(`javax.xml.bind.annotation.XmlRootElement`)で、
これが効かないというのだからJAXBが効いてないのだろうと思い、
そういえば`build.gradle`に`jackson-dataformat-xml`って書いたな、
というのを思い出し、
ZIP中でのXMLのmarshallingをJAXBのものでやるようにして
`build.gradle`から`jackson-dataformat-xml`を追い出したら、
元に戻りました。

JAXBでのmarshalling、ちょこっと面倒ですが、
`context = JAXBContext.newInstance(Bean.class)`して
`marshaller = context.createMarshaller()`作って、
`marshaller.marshal(bean, zipOutputStream)`すればいいんですね。