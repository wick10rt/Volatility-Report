conda activate vol3

vol -h

cd ~/memforensics

ls -lh OtterCTF.vmem

file OtterCTF.vmem

md5sum OtterCTF.vmem

vol -f ~/memforensics/OtterCTF.vmem windows.info

vol -f ~/memforensics/OtterCTF.vmem windows.psscan

vol -f ~/memforensics/OtterCTF.vmem windows.psscan | grep -Ei "vmware-tray|Rick And Morty|vmtoolsd|explorer|BitTorrent|LunarMS"

vol -f ~/memforensics/OtterCTF.vmem windows.netscan

mkdir -p ~/vol3-symbols

wget -P ~/vol3-symbols https://downloads.volatilityfoundation.org/volatility3/symbols/windows.zip

vol -s ~/vol3-symbols -f ~/memforensics/OtterCTF.vmem windows.info
