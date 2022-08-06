---
title:  "Bezier Curve with Unity 3D"
date:   2022-08-07 20:00:00
categories: [Unity]
tags: [Bezier Curve]
---
![image](https://user-images.githubusercontent.com/76097749/183246966-a9f9160b-f856-4019-82bf-7c42710c3583.png)

![image](https://user-images.githubusercontent.com/76097749/183246957-4116eb3b-a176-4aab-8b36-139a9d68d2b4.png)

If you use the Bezier curve principle to use traces of points on the Bezier curve, you can express a smooth curve.
<br><br>

If you put in a vector list, it's calculated from the code that solved the formula above traces of dots (dot B) come out.
<br><br>

A curve is drawn according to the change of t = [0,1].
## PhysicsUtility.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PhysicsUtility : MonoBehaviour
{ /// <summary>
  /// Combination
  /// </summary>
  /// <param name="n"></param>
  /// <param name="i"></param>
  /// <returns></returns>

    public static float Combination(int n, int i)
    {
        if (i < 0)
        {
            return 1;
        }
        else if (i > n)
        {
            return 1;
        }
        float p = 1;
        for (int j = 0; j < i; j++)
        {
            p *= (n - j);
        }
        float rP = 1;
        for (int k = 1; k <= i; k++)
        {
            rP *= k;
        }
        return p / rP;
    }
    /// <summary>
    /// Final interpolated position of the bezier curve <br/>
    /// </summary>
    /// <param name="dataset"></param>
    /// <param name="t"></param>
    /// <returns></returns>
    public static Vector3 BezierCurve(List<Vector3> dataset, float t)
    {
        if (dataset.Count <= 1)
        {
            throw new InvalidOperationException("You must include the number of at least two points!!");
        }
        int n = dataset.Count - 1;
        Vector3 B = new Vector3();
        for (int i = 0; i <= n; i++)
        {
            B += Combination(n, i) * Mathf.Pow(t, i) * Mathf.Pow((1 - t), n - i) * dataset[i];
        }
        return B;
    }
}

```

## MovingGround.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MovingGround : MonoBehaviour
{

    [Range(0, 1)]
    public float vTest = 0;
    public float speed = 0.01f;

    Rigidbody rigid;

    public List<Vector3> dataSets = new List<Vector3>();
    private void Awake()
    {
        rigid = GetComponent<Rigidbody>();
    }
    private void Start()
    {
        StartCoroutine(Move());
    }
    private IEnumerator Move()
    {
        while (true)
        {
            while (vTest < 1)
            {
                float sp = speed * Time.fixedDeltaTime;
                rigid.position = PhysicsUtility.BezierCurve(dataSets, vTest);
                vTest = Mathf.Clamp01(vTest + sp);
                yield return new WaitForFixedUpdate();
            }
            while (vTest > 0)
            {
                float sp = speed * Time.fixedDeltaTime;
                vTest = Mathf.Clamp01(vTest - sp);
                rigid.position = PhysicsUtility.BezierCurve(dataSets, vTest);
                yield return new WaitForFixedUpdate();
            }
        }
    }

}
```

## MovingEditor.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

[CanEditMultipleObjects]
[CustomEditor(typeof(MovingGround))]
public class MovingEditor : Editor
{
    private MovingGround Generator;
    SerializedProperty dataset;
    private void OnEnable()
    {
        Generator = target as MovingGround;
        dataset = serializedObject.FindProperty("dataSets");
    }

    private void OnSceneGUI()
    {
        if (Generator.dataSets.Count <= 1)
        {
            return;
        }

        EditorGUI.BeginChangeCheck();
        Queue<Vector3> buffer = new Queue<Vector3>();
        for (int i = 0; i < Generator.dataSets.Count; i++)
        {
            buffer.Enqueue(Handles.PositionHandle(Generator.dataSets[i], Quaternion.identity));
        }

        if (EditorGUI.EndChangeCheck())
        {
            Undo.RecordObject(Generator, $"Change {nameof(Generator.dataSets)}");

            for (int i = 0; i < Generator.dataSets.Count; i++)
            {
                Generator.dataSets[i] = buffer.Dequeue();
            }
            EditorUtility.SetDirty(Generator);
        }

        for (int i = 0; i < Generator.dataSets.Count - 1; i++)
        {
            Handles.DrawLine(Generator.dataSets[i], Generator.dataSets[i + 1]);
        }
        int detail = 50;
        for (float i = 0; i < detail; i++)
        {
            float value_before = i / detail;
            Vector3 before = PhysicsUtility.BezierCurve(Generator.dataSets, value_before);

            float value_after = (i + 1) / detail;
            Vector3 after = PhysicsUtility.BezierCurve(Generator.dataSets, value_after); ;


            Handles.DrawLine(before, after);
        }
    }
}

```

## Result Video

<video width="100%" height="100%" controls="controls">
  <source src="/Asset/Bezier/BezierCurve.mp4" type="video/mp4">
</video>

## References

https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%A7%80%EC%97%90_%EA%B3%A1%EC%84%A0

https://rito15.github.io/posts/unity-study-bezier-curve/

<b>Thank you :)</b>
