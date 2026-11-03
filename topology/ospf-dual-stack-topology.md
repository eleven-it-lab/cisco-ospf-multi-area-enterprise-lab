graph TD
%% Define Areas
subgraph Area0 [Area 0: Backbone Core]
R1((R1))
R2((R2 ABR))
R3((R3 ABR))
R4((R4 ABR))
end

    subgraph Area51 [Area 51: Standard NSSA]
        R5((R5 ASBR))
    end

    subgraph External [External Network]
        R6((R6 Branch))
    end

    subgraph Area2 [Area 2: Transit Area]
        R7((R7 Transit))
    end

    subgraph Area0_Isolated [Area 0: Discontiguous]
        R8((R8 Edge))
    end

    subgraph Area3 [Area 3: Legacy NBMA WAN]
        R9((R9 Spoke))
        R10((R10 Spoke))
    end

    %% Area 0 Links
    R1 <-->|Eth0/0 <br> 172.16.10.1| R2
    R1 <-->|Eth0/0 <br> 172.16.10.1| R3
    R1 <-->|Eth0/0 <br> 172.16.10.1| R4
    R2 <-->|Eth0/0 <br> 172.16.10.2| R3
    R3 <-->|Eth0/0 <br> 172.16.10.3| R4

    %% Area 51 Links
    R2 <-->|Eth0/1 <br> 192.168.2.1 <br> Cost: 500| R5
    R4 <-->|Eth0/1 <br> 192.168.4.1 <br> Cost: 10| R5
    R5 <-->|Eth0/2 <br> Ext| R6

    %% Area 2 & Virtual Link
    R3 <-->|Eth0/1 <br> 192.168.3.1| R7
    R3 -.->|OSPF Virtual Link <br> Transit: Area 2| R7
    R7 <-->|Eth0/2 <br> 192.168.7.1| R8

    %% Area 3 NBMA Hub & Spoke
    R3 <-->|Eth0/2 <br> 192.168.9.3| R9
    R3 <-->|Eth0/2 <br> 192.168.9.3| R10
    R9 <-->|192.168.9.9| R10

    classDef abr fill:#f9f,stroke:#333,stroke-width:2px;
    classDef asbr fill:#ff9,stroke:#333,stroke-width:2px;
    class R2,R3,R4,R7 abr;
    class R5 asbr;
