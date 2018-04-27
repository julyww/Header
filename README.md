# Header
        在Unity编辑器中增加脚本的中文说明
        http://www.mamicode.com/info-detail-1744538.html

        在游戏中，程序，美术，策划甚至音效都是分工合作的。很多时候，对于unity3d中一堆英文，大家都会看得很郁闷。尤其是不同的程序员，命名方式也不尽相同，甚至还是用拼音。因此，在脚本中增加一些中文显示，就能够很好地解决这个问题。

        首先，unity中对于字段(Field)已经有了很好的中文显示方法[Header]标签

        比如

using UnityEngine; 
public class TestScript : MonoBehaviour 
{ 
    [Header("变量A")] 
    public float A; 
}

   显示如下
技术分享
 

        但是对于[Header]，它本身不支持在非字段上做标签，所以想显示类的说明或者类函数的说明就无能为力了。

        既然如此，我们可以创建一个自定义的新标签MonoHeaderAttribute，它继承于Header，并且限制只能在类和函数上。当然，我们也可以不继承Header，完全也可以自己写一个仅继承于Attribute的标签。思路是一样的。这里用的是前者。

using System;
using UnityEngine;
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class MonoHeaderAttribute : HeaderAttribute
{
    public MonoHeaderAttribute(string header) : base(header)
    {}
}
 

        然后unity中创建一个editor目录，在editor目录下创建显示脚本MonoDescriptionEditor，继承于Editor，并定于CustomEditor标签为MonoBehaviour。注意，CustomEditor的第二个参数必须为true，表示对继承于MonoBehaviour的子类都有效。

        在其OnEnable事件中，通过反射获取到MonoHeaderAttribute的信息，然后在OnInspectorGUI显示出来。当然MonoDescriptionEditor完全可以自由定义，这里使用了HelpBox来显示。

using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(MonoBehaviour), true)]
public class MonoDescriptionEditor : Editor
{
    private string m_res;

    public void OnEnable()
    {
        m_res = "";
        var type = target.GetType();
        var atts = type.GetCustomAttributes(typeof(MonoHeaderAttribute), true);
        if (atts.Length <= 0)
            return;
        var att = (MonoHeaderAttribute)atts[0];
        m_res = att.header + "\n";
        var methods = type.GetMethods();
        foreach (var method in methods)
        {
            var matts = method.GetCustomAttributes(typeof(MonoHeaderAttribute), true);
            if (matts.Length > 0)
            {
                var matt = (MonoHeaderAttribute)matts[0];
                if (method.DeclaringType != null)
                    m_res += $"\n{method.DeclaringType.Name}.{method.Name}\n{matt.header}\n";
            }
        }
    }

    private static bool s_fold = true;
    public override void OnInspectorGUI()
    {
        base.OnInspectorGUI();
        if (string.IsNullOrEmpty(m_res))
            return;
        var color = GUI.color;
        GUI.contentColor = Color.cyan;
        s_fold = EditorGUILayout.Foldout(s_fold, "说明");
        if (s_fold)
            EditorGUILayout.HelpBox(m_res, MessageType.Info);
        GUI.contentColor = color;
    }
}
    
    回到TestScript，加入一些测试代码。在class上和一些函数上，加入了一些MonoHeader。
using UnityEngine;

[MonoHeader("这是一个测试脚本：TestScript")]
public class TestScript : MonoBehaviour
{
    internal void Start()
    {
    }

    [MonoHeader("这是一个测试函数")]
    public void TestA()
    {
    }

    [MonoHeader("第二个函数")]
    public void TestB()
    {
    }

    [Header("变量A")]
    public float A;
}
    

技术分享
         （PS：试过用MonoHeader替代Header来保证统一性，但是Unity对于字段只认Header，最终没有效果，故MonoHeader用于类和方法的说明，字段的说明用Header）

在Unity编辑器中增加脚本的中文说明

原文地址：http://www.cnblogs.com/CodeGize/p/6661507.html
