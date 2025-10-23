cd $HOME

rm -rf atomone

git clone https://github.com/atomone-hub/atomone

cd atomone

git checkout v2.1.0

make build

sudo systemctl restart atomoned

sudo journalctl -u atomoned -f
