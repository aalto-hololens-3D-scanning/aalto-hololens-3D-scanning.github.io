---
layout: default
title: Team
teamnames:
    - Seyoung Park

imageurls:
    - /images/seyoung.jpg

descriptions:
    - >
        Fourth-year BSc student in CS at Aalto University
    
customstyles:
    - team.css
---

<table id="team">
{% for name in page.teamnames %}
    <tr>
        <td>
            <h2 class="membername">{{ name }}</h2>
            <div class="memberdesc">{{ page.descriptions[forloop.index0] }}</div>
        </td>
        <td>
            <img class="memberimage" src="{{ page.imageurls[forloop.index0]}}" />
        </td>
    </tr>
{% endfor %}
</table>
