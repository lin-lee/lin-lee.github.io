---

# Header1

layout : post
title: "Object serialization and Deserialization"
date: 2017-06-01
categories: Blog

---

This article is talk about `RMI principle`, source code is `openjdk-7-fcs-src-b147-27_jun_2011.zip` .

Let start, no matter `RMI`,`Hessien`,`Dubbo` etc . I think they all usr Object serialization and deserialization. next is about rmi serialization
and deserialization.

 we talk about Object Serialization. so this is question about it. How Client Object Serialization ? and Server Obejct Deserialization ?
next is demo about serialization

Student.java




	import java.io.Serializable;
    
    public class Student implements Serializable {  
	  
    public static  String countryName="china";  
    private int id;  
    private String name;  
    private String sex;  
  
    public String getSex() {  
  
        return sex;  
    }  
  
    public void setSex(String sex) {  
        this.sex = sex;  
    }  
  
    public int getId() {  
        return id;  
    }  
  
    public void setId(int id) {  
        this.id = id;  
    }  
  
    @Override  
    public String toString() {  
        return "student{" +  
                "id=" + id +  
                ", name='" + name + '\'' +  
                '}';  
    }  
  
    public String getName() {  
        String s="adaf";  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }
  
    } 

Student is a Class to Transport from client to server. so the class must serialization and deserialization.

next is main method class

    
    

import java.io.*;

public class ObjectStreamDemo {
    private static final String filePath = "/home/lin/a.txt";

    public static void main(String[] args) {
        // writeObj();  
        readObj();
    }

    public static void writeObj() {
        Student s = new Student();
        s.setId(8);
        s.setName("张三");
        s.countryName = "USA";

        try {
            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(filePath));
            oos.writeObject(s);
            oos.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void readObj() {
        try {
            ObjectInputStream ooi = new ObjectInputStream(new FileInputStream(filePath));
            try {
                Object obj = ooi.readObject();
                Student s = (Student) obj;
                //  person s=(person)obj;       
                System.out.println("id:" + s.getId() + ",name:" + s.getName() + ",countryName:" + s.countryName);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
            ooi.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


when writeObject run, create a file ,let's see it.

`vim -b writeObject.result
::%!xxd
`

    0000000: aced 0005 7372 0007 5374 7564 656e 74a1  ....sr..Student.
    0000010: 640c a8a5 23ff d402 0003 4900 0269 644c  d...#.....I..idL
    0000020: 0004 6e61 6d65 7400 124c 6a61 7661 2f6c  ..namet..Ljava/l
    0000030: 616e 672f 5374 7269 6e67 3b4c 0003 7365  ang/String;L..se
    0000040: 7871 007e 0001 7870 0000 0008 7400 06e5  xq.~..xp....t...
    0000050: bca0 e4b8 8970                           .....p

next is writeObject.result.resolve

    aced 0005:  Magic number
    73 : Class
    72 : ClassDesc
    0007: Class length
    5374 7564 656e 74 Student
    a1 640c a8a5 23ff d4: suid
    02 : flags
    0003 : numFields
    49 : tcode  assign to first fields. ascii is I int type
    00 02: 即 表示 field length is 2
    69 64: id field name
    4c : L second field tcode ,Class type
    0004: second field length is 4
    6e61 6d65: name second field name
    74: field class Type TC_STRING
    0012: field signature length
    4c 6a61 7661 2f6c 616e 672f 5374 7269 6e67 3b : Ljava/lang/String;

    4c : the same as above
    0003 : field length is 3
    7365 78 : sex
    71: field class Type TC_REFERENCE
    007e 0001 : reference to Ljava/lang/String
    78 :  Attemps to read in the next block data header (if any)
      then TC_ENDBLOCKDATA
    70 : TC_NULL superDesc is null
    0000 0008 : id value
    74 : TC_STRING
    00 06 length 6
    e5 bca0 e4b8 89 : 张三
    70 : TC_NULL

